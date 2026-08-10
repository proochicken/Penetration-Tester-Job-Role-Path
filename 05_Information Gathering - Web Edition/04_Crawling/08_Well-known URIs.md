# Well-known URIs

## Ý chính

`.well-known` là thư mục chuẩn tại root domain của website, được định nghĩa trong RFC 8615.

Đường dẫn:

```text
https://example.com/.well-known/
```
Nó dùng để chứa metadata/configuration liên quan đến service, protocol và security mechanism của website.
## Vì sao quan trọng?

`.well-known` giúp browser, app, client và security tool tự động tìm cấu hình ở vị trí chuẩn.

Ví dụ:
```
https://example.com/.well-known/security.txt
https://example.com/.well-known/openid-configuration
https://example.com/.well-known/assetlinks.json
https://example.com/.well-known/mta-sts.txt
```

## Một số URI đáng chú ý

|URI|Mục đích|
|---|---|
|`security.txt`|Thông tin liên hệ để báo cáo lỗ hổng|
|`change-password`|URL chuẩn để đổi mật khẩu|
|`openid-configuration`|Cấu hình OpenID Connect/OAuth|
|`assetlinks.json`|Xác minh liên kết domain với app|
|`mta-sts.txt`|Chính sách bảo mật SMTP MTA-STS|

## Web Recon với `.well-known`

Trong recon, cần kiểm tra `.well-known` vì nó có thể lộ endpoint/config quan trọng.

Endpoint đặc biệt quan trọng:

```
/.well-known/openid-configuration
```

Nó có thể trả về JSON chứa:

```
{  "issuer": "https://example.com",  "authorization_endpoint": "https://example.com/oauth2/authorize",  "token_endpoint": "https://example.com/oauth2/token",  "userinfo_endpoint": "https://example.com/oauth2/userinfo",  "jwks_uri": "https://example.com/oauth2/jwks",  "response_types_supported": ["code", "token", "id_token"],  "scopes_supported": ["openid", "profile", "email"],  "id_token_signing_alg_values_supported": ["RS256"]}
```

## Thông tin cần chú ý trong `openid-configuration`

- `issuer`: Identity Provider
- `authorization_endpoint`: endpoint login/authorize
- `token_endpoint`: endpoint cấp token
- `userinfo_endpoint`: endpoint lấy thông tin user
- `jwks_uri`: public keys để verify JWT
- `response_types_supported`: flow được hỗ trợ
- `scopes_supported`: quyền/scope được hỗ trợ
- `id_token_signing_alg_values_supported`: thuật toán ký token

## Ghi nhớ

`.well-known` không phải exploit. Nó là nguồn recon có cấu trúc, giúp tìm endpoint và cấu hình quan trọng.

Đặc biệt cần chú ý:

- `security.txt`
- `openid-configuration`
- `jwks_uri`
- `assetlinks.json`
- `mta-sts.txt`

Sau khi tìm được endpoint/config, cần phân tích xem có misconfiguration hay rủi ro security không.

# Well-known URI Recon Checklist  
## 1. Xác định target  
  
- [ ] Xác định domain/IP trong scope.  
- [ ] Nếu là HTB/lab, thêm hostname vào `/etc/hosts` nếu cần.  
- [ ] Xác định scheme/port: HTTP hay HTTPS.  
  
Ví dụ:  
  
```text  
https://target.com  
http://target.htb
```
## 2. Kiểm tra thư mục `.well-known`

```
curl -i https://target.com/.well-known/
```

Ghi lại:

- [ ]  Status code
- [ ]  Directory listing có bật không
- [ ]  Có redirect không
- [ ]  Có file nào được list không

## 3. Kiểm tra `security.txt`

```
curl -i https://target.com/.well-known/security.txt
```

Ghi lại:

- [ ]  Contact
- [ ]  Policy
- [ ]  Preferred-Languages
- [ ]  Encryption
- [ ]  Expires

## 4. Kiểm tra OpenID Connect Discovery

```
curl -s https://target.com/.well-known/openid-configuration | jq
```

Nếu có JSON, ghi lại:

- [ ]  `issuer`
- [ ]  `authorization_endpoint`
- [ ]  `token_endpoint`
- [ ]  `userinfo_endpoint`
- [ ]  `jwks_uri`
- [ ]  `scopes_supported`
- [ ]  `response_types_supported`
- [ ]  `id_token_signing_alg_values_supported`

## 5. Kiểm tra JWKS

Nếu có `jwks_uri`:

```
curl -s https://target.com/oauth2/jwks | jq
```

Ghi lại:

- [ ]  `kid`
- [ ]  `kty`
- [ ]  `alg`
- [ ]  `use`
- [ ]  Số lượng keys
- [ ]  Có key cũ/không dùng nữa không

## 6. Kiểm tra `assetlinks.json`

```
curl -s https://target.com/.well-known/assetlinks.json | jq
```

Ghi lại:

- [ ]  Android package name
- [ ]  SHA256 certificate fingerprints
- [ ]  App/domain relationship

## 7. Kiểm tra `mta-sts.txt`

```
curl -i https://mta-sts.target.com/.well-known/mta-sts.txt
```

Hoặc nếu theo domain hiện tại:

```
curl -i https://target.com/.well-known/mta-sts.txt
```

Ghi lại:

- [ ]  `version`
- [ ]  `mode`
- [ ]  `mx`
- [ ]  `max_age`

## 8. Kiểm tra `change-password`

```
curl -I https://target.com/.well-known/change-password
```

Ghi lại:

- [ ]  Có redirect đến trang đổi mật khẩu không?
- [ ]  Có lộ endpoint account management không?

## 9. Fuzz thêm `.well-known`

Nếu scope cho phép:

```
ffuf -u https://target.com/.well-known/FUZZ \  -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Tìm:

- [ ]  File config
- [ ]  JSON metadata
- [ ]  Endpoint auth
- [ ]  App/mobile association files

## 10. Phân tích security

- [ ]  Endpoint nào liên quan authentication?
- [ ]  Có OAuth/OIDC misconfiguration không?
- [ ]  JWKS có bất thường không?
- [ ]  Scope/response type có rủi ro không?
- [ ]  Có thông tin contact/security policy không?
- [ ]  Có file public không nên public không?

## 11. Ghi chú report

- [ ]  URI phát hiện được
- [ ]  Nội dung quan trọng
- [ ]  Impact thực tế
- [ ]  Bằng chứng
- [ ]  Không report chỉ vì endpoint tồn tại; cần chỉ ra rủi ro/misconfig