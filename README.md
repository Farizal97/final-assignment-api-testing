# Final Assignment API Testing - AfterOffice

Final Assignment API Testing menggunakan **Postman**, **Data-Driven Testing (CSV)**, dan **GitHub Actions**.

Project ini melakukan automated API testing terhadap:

https://api-script-labs.hendri.me

## Tools

- Postman
- Postman CLI
- CSV Data-Driven Testing
- GitHub
- GitHub Actions

## API Test Scenario

Collection terdiri dari 11 request:

1. POST - Login
2. GET - Get All Labs
3. POST - Create Lab
4. GET - Get Lab by ID
5. PUT - Update Lab
6. GET - Get After Update
7. DELETE - Delete Lab
8. GET - Get After Delete
9. POST - Invalid Data
10. POST - Edge Case
11. POST - CSV Data Drive

Testing mencakup operasi CRUD pada endpoint `/api/labs` serta autentikasi melalui `/api/auth/login`.

## Authentication & Token Handling

Token diperoleh secara otomatis dari response Login.

Token kemudian disimpan ke Collection Variable:

`auth_secret_0jl8`

Seluruh request `/api/labs` menggunakan Bearer Token dari variable tersebut sehingga token tidak di-hardcode secara manual pada setiap request.

## Dynamic Lab ID

Setelah request Create Lab berhasil, ID lab disimpan ke Collection Variable:

`lab_id`

Variable tersebut digunakan oleh request GET, PUT, dan DELETE sehingga pengujian CRUD dapat berjalan secara berurutan dan otomatis.

## Test Assertions

Setiap request memiliki automated test/assertion, meliputi:

- Status code
- Response body
- Response time
- Validasi hasil response
- Validasi token
- Validasi ID resource

## Data-Driven Testing

Data-driven testing menggunakan file:

`data_test.csv`

CSV terdiri dari minimal 3 skenario:

- Valid
- Invalid
- Edge Case

Contoh struktur:

```csv
title,description
CSV Valid Test 001,Valid data from CSV
,Invalid test empty title
Z,Edge case one character title
