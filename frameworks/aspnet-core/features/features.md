# ASP.NET Core — Features

---

## 🗄️ Caché

```csharp
// IMemoryCache — caché en memoria del proceso
builder.Services.AddMemoryCache();

public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _productRepository;

    public async Task<ProductResponse> FindByIdAsync(Guid id)
    {
        return await _cache.GetOrCreateAsync($"product:{id}", async entry =>
        {
            entry.SlidingExpiration = TimeSpan.FromMinutes(10);
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1);

            var product = await _productRepository.FindByIdAsync(id)
                ?? throw new ResourceNotFoundException("Product", id);
            return product.Adapt<ProductResponse>();
        }) ?? throw new ResourceNotFoundException("Product", id);
    }

    public void InvalidateCache(Guid id) => _cache.Remove($"product:{id}");
}

// IDistributedCache con Redis — caché compartida entre instancias
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MiApi:";
});

public class ProductService
{
    private readonly IDistributedCache _cache;

    public async Task<ProductResponse> FindByIdAsync(Guid id)
    {
        var cacheKey = $"product:{id}";
        var cached = await _cache.GetStringAsync(cacheKey);

        if (cached != null)
            return JsonSerializer.Deserialize<ProductResponse>(cached)!;

        var product = await _productRepository.FindByIdAsync(id)
            ?? throw new ResourceNotFoundException("Product", id);
        var response = product.Adapt<ProductResponse>();

        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(response),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
            });

        return response;
    }

    public async Task InvalidateCacheAsync(Guid id) =>
        await _cache.RemoveAsync($"product:{id}");
}

// HybridCache (.NET 9+) — combina caché local + distribuida automáticamente
builder.Services.AddHybridCache();

public class ProductService
{
    private readonly HybridCache _cache;

    public async Task<ProductResponse> FindByIdAsync(Guid id)
    {
        return await _cache.GetOrCreateAsync(
            $"product:{id}",
            async cancel => await LoadProductAsync(id, cancel),
            new HybridCacheEntryOptions { Expiration = TimeSpan.FromHours(1) });
    }
}
```

---

## ⚡ Tareas en segundo plano

```csharp
// BackgroundService — tarea de larga duración en segundo plano
public class InventorySyncService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<InventorySyncService> _logger;

    public InventorySyncService(IServiceScopeFactory scopeFactory, ILogger<InventorySyncService> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();   // DbContext es Scoped
            var repository = scope.ServiceProvider.GetRequiredService<IProductRepository>();

            try
            {
                _logger.LogInformation("Sincronizando inventario...");
                // ... lógica de sincronización ...
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error al sincronizar inventario");
            }

            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}

// Registro: builder.Services.AddHostedService<InventorySyncService>();

// Colas en memoria para trabajo diferido (fire-and-forget desde un endpoint)
public interface IBackgroundTaskQueue
{
    void QueueBackgroundWorkItem(Func<CancellationToken, Task> workItem);
    Task<Func<CancellationToken, Task>> DequeueAsync(CancellationToken cancellationToken);
}

public class BackgroundTaskQueue : IBackgroundTaskQueue
{
    private readonly Channel<Func<CancellationToken, Task>> _queue =
        Channel.CreateUnbounded<Func<CancellationToken, Task>>();

    public void QueueBackgroundWorkItem(Func<CancellationToken, Task> workItem) =>
        _queue.Writer.TryWrite(workItem);

    public async Task<Func<CancellationToken, Task>> DequeueAsync(CancellationToken cancellationToken) =>
        await _queue.Reader.ReadAsync(cancellationToken);
}

public class QueuedHostedService : BackgroundService
{
    private readonly IBackgroundTaskQueue _taskQueue;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var workItem = await _taskQueue.DequeueAsync(stoppingToken);
            await workItem(stoppingToken);
        }
    }
}
```

---

## ⏱️ Scheduling

```csharp
// Quartz.NET — scheduling robusto con soporte de expresiones cron
public class CleanupJob : IJob
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<CleanupJob> _logger;

    public CleanupJob(IServiceScopeFactory scopeFactory, ILogger<CleanupJob> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    public async Task Execute(IJobExecutionContext context)
    {
        _logger.LogInformation("Limpiando datos antiguos...");
        using var scope = _scopeFactory.CreateScope();
        // ... lógica de limpieza ...
    }
}

// Registro en Program.cs
builder.Services.AddQuartz(q =>
{
    var jobKey = new JobKey("CleanupJob");
    q.AddJob<CleanupJob>(opts => opts.WithIdentity(jobKey));

    q.AddTrigger(opts => opts
        .ForJob(jobKey)
        .WithIdentity("CleanupJob-trigger")
        .WithCronSchedule("0 0 2 * * ?"));   // Cada día a las 2:00 AM
});
builder.Services.AddQuartzHostedService(options => options.WaitForJobsToComplete = true);

// Expresiones cron comunes (formato Quartz — 6 u 7 campos)
// "0 * * * * ?"        — cada minuto
// "0 */5 * * * ?"      — cada 5 minutos
// "0 0 * * * ?"        — cada hora
// "0 0 0 * * ?"        — cada día a medianoche
// "0 0 12 * * ?"       — cada día al mediodía
// "0 0 0 ? * MON"      — todos los lunes a medianoche
// "0 0 0 1 * ?"        — primero de cada mes a medianoche

// PeriodicTimer — alternativa simple sin dependencias externas
public class SimpleSchedulerService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var timer = new PeriodicTimer(TimeSpan.FromMinutes(5));
        while (await timer.WaitForNextTickAsync(stoppingToken))
        {
            // ... tarea periódica ...
        }
    }
}
```

---

## 📢 Eventos — MediatR

```csharp
// MediatR tiene licencia dual RPL-1.5 / comercial desde la v13 (jul 2025) — gratuita para
// individuos y empresas con facturación anual menor a USD 5M (ver mediatr.io)

// Evento de dominio (notificación)
public record ProductCreatedEvent(Guid ProductId, string Sku, string Category) : INotification;

// Publicar eventos desde el Service
public class ProductService : IProductService
{
    private readonly IMediator _mediator;
    private readonly IProductRepository _productRepository;

    public async Task<ProductResponse> CreateAsync(CreateProductRequest request)
    {
        var product = request.Adapt<Product>();
        await _productRepository.SaveAsync(product);

        // Publica el evento — se procesan todos los handlers registrados
        await _mediator.Publish(new ProductCreatedEvent(
            product.Id, product.Sku, product.Category.Name));

        return product.Adapt<ProductResponse>();
    }
}

// Handlers — múltiples handlers pueden reaccionar al mismo evento
public class SendWelcomeEmailHandler : INotificationHandler<ProductCreatedEvent>
{
    private readonly INotificationService _notificationService;

    public async Task Handle(ProductCreatedEvent notification, CancellationToken cancellationToken)
    {
        await _notificationService.NotifyNewProductAsync(notification.ProductId);
    }
}

public class IndexProductHandler : INotificationHandler<ProductCreatedEvent>
{
    private readonly ISearchIndexService _searchIndexService;

    public async Task Handle(ProductCreatedEvent notification, CancellationToken cancellationToken)
    {
        await _searchIndexService.IndexAsync(notification.ProductId);
    }
}

// Registro: builder.Services.AddMediatR(cfg =>
//     cfg.RegisterServicesFromAssemblyContaining<Program>());

// MediatR también soporta el patrón CQRS con IRequest / IRequestHandler
public record CreateProductCommand(CreateProductRequest Request) : IRequest<ProductResponse>;

public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, ProductResponse>
{
    private readonly IProductRepository _productRepository;

    public async Task<ProductResponse> Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        var product = request.Request.Adapt<Product>();
        await _productRepository.SaveAsync(product);
        return product.Adapt<ProductResponse>();
    }
}

// Uso en el endpoint: await mediator.Send(new CreateProductCommand(request));
```

---

## 📧 Mail

```csharp
// MailKit — envío de correos con soporte HTML y adjuntos
public class MailService : IMailService
{
    private readonly IOptions<MailOptions> _options;
    private readonly ILogger<MailService> _logger;

    public async Task SendSimpleAsync(string to, string subject, string body)
    {
        var message = new MimeMessage();
        message.From.Add(MailboxAddress.Parse(_options.Value.From));
        message.To.Add(MailboxAddress.Parse(to));
        message.Subject = subject;
        message.Body = new TextPart("plain") { Text = body };

        await SendAsync(message);
    }

    public async Task SendHtmlAsync(string to, string subject, string htmlBody, List<string>? attachmentPaths = null)
    {
        var message = new MimeMessage();
        message.From.Add(MailboxAddress.Parse(_options.Value.From));
        message.To.Add(MailboxAddress.Parse(to));
        message.Subject = subject;

        var builder = new BodyBuilder { HtmlBody = htmlBody };
        if (attachmentPaths != null)
        {
            foreach (var path in attachmentPaths)
                builder.Attachments.Add(path);
        }
        message.Body = builder.ToMessageBody();

        await SendAsync(message);
    }

    private async Task SendAsync(MimeMessage message)
    {
        try
        {
            using var client = new SmtpClient();
            await client.ConnectAsync(_options.Value.Host, _options.Value.Port, SecureSocketOptions.StartTls);
            await client.AuthenticateAsync(_options.Value.Username, _options.Value.Password);
            await client.SendAsync(message);
            await client.DisconnectAsync(true);
            _logger.LogInformation("Email enviado a: {To}", message.To);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error al enviar email");
        }
    }
}

public class MailOptions
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; } = 587;
    public string From { get; set; } = string.Empty;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

---

## 🩺 Health Checks

```csharp
// Registro de health checks
builder.Services.AddHealthChecks()
    .AddNpgSql(builder.Configuration.GetConnectionString("Default")!, name: "postgresql")
    .AddRedis(builder.Configuration.GetConnectionString("Redis")!, name: "redis")
    .AddCheck<ExternalApiHealthCheck>("external-api");

// Health check personalizado
public class ExternalApiHealthCheck : IHealthCheck
{
    private readonly HttpClient _httpClient;

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _httpClient.GetAsync("https://api.externa.com/health", cancellationToken);
            return response.IsSuccessStatusCode
                ? HealthCheckResult.Healthy("API externa disponible")
                : HealthCheckResult.Degraded("API externa respondió con error");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("API externa no disponible", ex);
        }
    }
}

// Endpoint con respuesta detallada en JSON
app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                description = e.Value.Description
            })
        });
        await context.Response.WriteAsync(result);
    }
});

// Endpoints separados para liveness / readiness (Kubernetes)
app.MapHealthChecks("/health/live", new HealthCheckOptions { Predicate = _ => false });
app.MapHealthChecks("/health/ready", new HealthCheckOptions { Predicate = check => check.Tags.Contains("ready") });
```

