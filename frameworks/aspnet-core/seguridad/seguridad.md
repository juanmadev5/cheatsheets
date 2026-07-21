# ASP.NET Core — Seguridad

---

## 🔐 Autenticación — JWT Bearer

```csharp
// JwtService — generación y validación de tokens
public class JwtService
{
    private readonly JwtOptions _options;

    public JwtService(IOptions<JwtOptions> options)
    {
        _options = options.Value;
    }

    public string GenerateToken(ApplicationUser user, IList<string> roles)
    {
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new(JwtRegisteredClaimNames.Email, user.Email!),
            new(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };
        claims.AddRange(roles.Select(role => new Claim(ClaimTypes.Role, role)));

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_options.Secret));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _options.Issuer,
            audience: _options.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_options.ExpirationMinutes),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public string GenerateRefreshToken()
    {
        var randomBytes = RandomNumberGenerator.GetBytes(64);
        return Convert.ToBase64String(randomBytes);
    }
}

// AuthService
public class AuthService : IAuthService
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly JwtService _jwtService;
    private readonly ILogger<AuthService> _logger;

    public async Task<AuthResponse> RegisterAsync(RegisterRequest request)
    {
        var existing = await _userManager.FindByEmailAsync(request.Email);
        if (existing != null)
            throw new DuplicateResourceException("El email ya está registrado");

        var user = new ApplicationUser
        {
            UserName = request.Email,
            Email = request.Email,
            Name = request.Name
        };

        var result = await _userManager.CreateAsync(user, request.Password);
        if (!result.Succeeded)
            throw new BusinessException("REGISTRATION_FAILED",
                string.Join(", ", result.Errors.Select(e => e.Description)));

        await _userManager.AddToRoleAsync(user, "User");
        _logger.LogInformation("Usuario registrado: {Email}", user.Email);

        return await BuildAuthResponseAsync(user);
    }

    public async Task<AuthResponse> LoginAsync(LoginRequest request)
    {
        var user = await _userManager.FindByEmailAsync(request.Email)
            ?? throw new ResourceNotFoundException("User", request.Email);

        var result = await _signInManager.CheckPasswordSignInAsync(
            user, request.Password, lockoutOnFailure: true);

        if (!result.Succeeded)
            throw new BusinessException("INVALID_CREDENTIALS", "Credenciales inválidas");

        _logger.LogInformation("Login exitoso: {Email}", user.Email);
        return await BuildAuthResponseAsync(user);
    }

    private async Task<AuthResponse> BuildAuthResponseAsync(ApplicationUser user)
    {
        var roles = await _userManager.GetRolesAsync(user);
        var accessToken = _jwtService.GenerateToken(user, roles);
        var refreshToken = _jwtService.GenerateRefreshToken();

        return new AuthResponse(accessToken, refreshToken, 3600, user.Adapt<UserResponse>());
    }
}

// AuthEndpoints (Minimal APIs)
public static class AuthEndpoints
{
    public static void MapAuthEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/api/v1/auth").WithTags("Auth");

        group.MapPost("/register", async (RegisterRequest request, IAuthService authService) =>
            Results.Created("/api/v1/auth/me", await authService.RegisterAsync(request)));

        group.MapPost("/login", async (LoginRequest request, IAuthService authService) =>
            Results.Ok(await authService.LoginAsync(request)));

        group.MapGet("/me", async (ClaimsPrincipal user, IAuthService authService) =>
            Results.Ok(await authService.GetCurrentUserAsync(user)))
            .RequireAuthorization();
    }
}
```

---

## 👤 ASP.NET Core Identity

```csharp
// Entidad de usuario personalizada
public class ApplicationUser : IdentityUser<Guid>
{
    public required string Name { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}

// DbContext con Identity
public class AppDbContext : IdentityDbContext<ApplicationUser, IdentityRole<Guid>, Guid>
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);   // Necesario para las tablas de Identity
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}

// Registro con configuración de contraseñas y lockout
builder.Services.AddIdentityCore<ApplicationUser>(options =>
    {
        options.Password.RequiredLength = 8;
        options.Password.RequireDigit = true;
        options.Password.RequireUppercase = true;
        options.Lockout.MaxFailedAccessAttempts = 5;
        options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
        options.User.RequireUniqueEmail = true;
    })
    .AddRoles<IdentityRole<Guid>>()
    .AddEntityFrameworkStores<AppDbContext>()
    .AddSignInManager()
    .AddDefaultTokenProviders();

// Seed de roles al iniciar la aplicación
using (var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole<Guid>>>();
    foreach (var role in new[] { "Admin", "User", "Moderator" })
    {
        if (!await roleManager.RoleExistsAsync(role))
            await roleManager.CreateAsync(new IdentityRole<Guid>(role));
    }
}
```

---

## 🔓 Autorización

```csharp
// Autorización basada en roles y políticas — configuración
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"))
    .AddPolicy("AdminOrModerator", policy => policy.RequireRole("Admin", "Moderator"))
    .AddPolicy("MinimumAge18", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "age") &&
            int.Parse(context.User.FindFirst("age")!.Value) >= 18));

// Uso en Minimal APIs
group.MapPost("/", CreateProduct).RequireAuthorization("AdminOnly");
group.MapGet("/", FindAll).RequireAuthorization();   // Solo requiere estar autenticado

// Uso en Controllers
[Authorize(Policy = "AdminOnly")]
[HttpDelete("{id:guid}")]
public async Task<IActionResult> Delete(Guid id) { /* ... */ }

[Authorize(Roles = "Admin,Moderator")]
[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateProductRequest req) { /* ... */ }

// Handler de autorización personalizado — lógica compleja (ej: dueño del recurso)
public class ProductOwnerRequirement : IAuthorizationRequirement { }

public class ProductOwnerHandler : AuthorizationHandler<ProductOwnerRequirement, Product>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        ProductOwnerRequirement requirement,
        Product resource)
    {
        var userId = context.User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
        if (context.User.IsInRole("Admin") || resource.CreatedBy == userId)
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

// Registro: builder.Services.AddScoped<IAuthorizationHandler, ProductOwnerHandler>();

// Uso manual con IAuthorizationService
public class ProductController : ControllerBase
{
    private readonly IAuthorizationService _authorizationService;

    [HttpPut("{id:guid}")]
    public async Task<IActionResult> Update(Guid id, UpdateProductRequest request)
    {
        var product = await _productService.FindEntityByIdAsync(id);
        var authResult = await _authorizationService.AuthorizeAsync(
            User, product, new ProductOwnerRequirement());

        if (!authResult.Succeeded) return Forbid();

        return Ok(await _productService.UpdateAsync(id, request));
    }
}

// Obtener el usuario autenticado
public class ProductController : ControllerBase
{
    [HttpGet("my-products")]
    [Authorize]
    public async Task<IActionResult> MyProducts()
    {
        var userId = Guid.Parse(User.FindFirst(JwtRegisteredClaimNames.Sub)!.Value);
        return Ok(await _productService.FindByUserAsync(userId));
    }
}
```

