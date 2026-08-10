# Certificate Transparency Logs

## Ý chính

Certificate Transparency Logs - CT Logs là các sổ ghi công khai, append-only, lưu lại SSL/TLS certificates đã được cấp.

Khi một CA cấp certificate mới, certificate/pre-certificate sẽ được submit vào nhiều CT logs. Các log này giúp phát hiện certificate giả mạo, certificate cấp sai và tăng tính minh bạch của Web PKI.

## Vì sao CT Logs quan trọng?

CT Logs giúp:

- Phát hiện rogue/mis-issued certificates
- Buộc Certificate Authority chịu trách nhiệm
- Tăng độ tin cậy của Web PKI
- Hỗ trợ website owner/security researcher monitor certificate đáng ngờ
- Hỗ trợ pentester tìm subdomain trong web recon

## Quy trình CT Logs

1. Website owner xin SSL/TLS certificate từ CA.
2. CA xác minh domain ownership và tạo pre-certificate.
3. CA submit pre-certificate vào CT logs.
4. CT log trả về SCT - Signed Certificate Timestamp.
5. SCT được đưa vào certificate cuối cùng.
6. Browser kiểm tra SCT khi người dùng truy cập website.
7. Security researcher/website owner monitor CT logs để tìm certificate bất thường.

## Merkle Tree

CT Logs dùng Merkle Tree để đảm bảo dữ liệu không bị sửa.

- Leaf node: certificate
- Intermediate node: hash của node con
- Merkle root: hash đại diện toàn bộ log
- Merkle path: bằng chứng certificate nằm trong log

Nếu certificate hoặc log bị sửa, root hash sẽ thay đổi.

## CT Logs trong Web Recon

CT logs rất hữu ích để tìm subdomain vì certificate thường chứa SAN entries.

Ví dụ SAN có thể lộ:

```text
www.example.com
api.example.com
dev.example.com
staging.example.com
admin.example.com
```

Ưu điểm so với brute-force:

- Không phụ thuộc wordlist
- Có thể tìm subdomain khó đoán
- Có thể thấy subdomain cũ/expired
- Ít tương tác trực tiếp với target

Nhược điểm:

- Dữ liệu có thể cũ
- Subdomain tìm được chưa chắc còn sống
- Cần validate lại bằng DNS/HTTP

## Tools

|Tool|Mục đích|
|---|---|
|`crt.sh`|Tìm certificate/subdomain nhanh từ CT logs|
|`Censys`|Tìm kiếm nâng cao certificate, IP, host, service|

## Command crt.sh API

```
curl -s "https://crt.sh/?q=facebook.com&output=json" \| jq -r '.[] | select(.name_value | contains("dev")) | .name_value' \| sort -u
```

Ý nghĩa:

- `curl`: lấy dữ liệu JSON từ crt.sh
- `jq`: parse/filter JSON
- `contains("dev")`: chỉ lấy subdomain chứa chuỗi `dev`
- `.name_value`: lấy domain/subdomain trong certificate
- `sort -u`: sort và loại trùng

## Ghi nhớ

CT logs là nguồn passive recon rất mạnh. Khi tìm subdomain, nên kết hợp:

1. CT logs
2. DNS enumeration
3. Subdomain brute-force
4. VHost fuzzing
5. HTTP validation