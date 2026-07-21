# Flutter — Seguridad

---

## 🔒 Seguridad

```yaml
dependencies:
  flutter_secure_storage: ^10.3.1
  local_auth: ^3.0.2
```

```dart
// flutter_secure_storage — tokens y datos sensibles
class SecureStorage {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(),  // default: RSA OAEP + AES-GCM (encryptedSharedPreferences quedó obsoleto)
    iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
  );

  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';
  static const _userIdKey = 'user_id';

  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _accessTokenKey, value: accessToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }

  Future<String?> getAccessToken() => _storage.read(key: _accessTokenKey);
  Future<String?> getRefreshToken() => _storage.read(key: _refreshTokenKey);

  Future<void> clearSession() async {
    await Future.wait([
      _storage.delete(key: _accessTokenKey),
      _storage.delete(key: _refreshTokenKey),
      _storage.delete(key: _userIdKey),
    ]);
  }
}

// Autenticación biométrica
class BiometricService {
  final _auth = LocalAuthentication();

  Future<bool> isAvailable() async {
    final canCheck = await _auth.canCheckBiometrics;
    final isDeviceSupported = await _auth.isDeviceSupported();
    return canCheck && isDeviceSupported;
  }

  Future<List<BiometricType>> getAvailableBiometrics() =>
      _auth.getAvailableBiometrics();

  Future<bool> authenticate({
    String reason = 'Confirmá tu identidad para continuar',
  }) async {
    try {
      return await _auth.authenticate(
        localizedReason: reason,
        options: const AuthenticationOptions(
          biometricOnly: false,     // Permitir PIN como fallback
          stickyAuth: true,
        ),
      );
    } on PlatformException {
      return false;
    }
  }

  Future<void> cancelAuthentication() => _auth.stopAuthentication();
}

// Certificate pinning con Dio
class PinningInterceptor extends Interceptor {
  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    // Implementar con dart:io HttpClient para pinning real en producción
    handler.next(response);
  }
}

// Configurar HttpClient con pinning
HttpClient createPinnedClient(List<String> pinnedCertificates) {
  final context = SecurityContext(withTrustedRoots: false);
  for (final cert in pinnedCertificates) {
    context.setTrustedCertificatesBytes(base64Decode(cert));
  }
  return HttpClient(context: context);
}

// Ofuscación — agregar en pubspec.yaml
// flutter build apk --obfuscate --split-debug-info=build/debug-info
// flutter build ipa --obfuscate --split-debug-info=build/debug-info
```


