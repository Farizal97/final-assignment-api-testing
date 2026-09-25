# Final Assignment API Testing - AfterOffice

Final Assignment API Testing menggunakan **Postman**, **Postman CLI**, **Data-Driven Testing (CSV)**, dan **GitHub Actions**.

Project ini dibuat untuk melakukan automated API testing terhadap endpoint Labs API dengan mencakup proses autentikasi, CRUD, automatic token handling, dynamic ID, test assertion, data-driven testing, dan CI/CD menggunakan GitHub Actions.

## Tools

Tools yang digunakan:

- Postman
- Postman CLI
- CSV Data-Driven Testing
- GitHub
- GitHub Actions

## API Endpoint

Base URL yang digunakan:

```text
https://api-script-labs.hendri.me
```

Endpoint utama:

```text
POST /api/auth/login
GET /api/labs
POST /api/labs
GET /api/labs/{id}
PUT /api/labs/{id}
DELETE /api/labs/{id}
```

## API Test Scenarios

Collection terdiri dari **11 request**:

1. `01 - Login`
2. `02 - GET All Labs`
3. `03 - POST Create Lab`
4. `04 - GET Lab by ID`
5. `05 - PUT Update Lab`
6. `06 - GET After Update`
7. `07 - DELETE Lab`
8. `08 - GET After Delete`
9. `09 - POST Invalid`
10. `10 - POST Edge Case`
11. `11 - POST CSV Data Drive`

Testing mencakup operasi CRUD pada endpoint `/api/labs` serta autentikasi melalui endpoint `/api/auth/login`.

## Authentication & Token Handling

Proses autentikasi dilakukan melalui:

```text
POST /api/auth/login
```

Token dari response login disimpan secara otomatis ke variable Postman.

Seluruh request `/api/labs` kemudian menggunakan **Bearer Token** dari variable tersebut.

Dengan mekanisme ini, token tidak perlu di-hardcode secara manual pada setiap request.

Alur autentikasi:

```text
Login
  ↓
Token diperoleh
  ↓
Token disimpan ke variable
  ↓
Bearer Token digunakan pada request /api/labs
```

## Dynamic Lab ID

Setelah request `POST Create Lab` berhasil, ID dari lab yang baru dibuat disimpan ke variable:

```text
lab_id
```

Variable tersebut kemudian digunakan oleh request berikutnya:

```text
GET Lab by ID
      ↓
PUT Update Lab
      ↓
GET After Update
      ↓
DELETE Lab
      ↓
GET After Delete
```

Dengan dynamic ID, proses pengujian CRUD dapat berjalan secara berurutan tanpa menggunakan ID yang di-hardcode.

## Test Assertions

Setiap request memiliki automated test/assertion.

Assertion yang digunakan mencakup:

- Status code
- Response body
- Response time
- Validasi keberhasilan response
- Validasi token
- Validasi ID resource

Contoh assertion status code:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

Contoh assertion response time:

```javascript
pm.test("Response time di bawah 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

## CRUD Testing

Pengujian CRUD dilakukan secara berurutan.

### Create

```text
POST /api/labs
```

Membuat data lab baru dan menyimpan ID hasil create ke variable `lab_id`.

### Read

```text
GET /api/labs
GET /api/labs/{{lab_id}}
```

Digunakan untuk mengambil seluruh data dan mengambil data berdasarkan ID.

### Update

```text
PUT /api/labs/{{lab_id}}
```

Mengubah data lab yang sebelumnya dibuat.

### Delete

```text
DELETE /api/labs/{{lab_id}}
```

Menghapus data lab.

Setelah proses delete dilakukan pengecekan kembali menggunakan:

```text
GET /api/labs/{{lab_id}}
```

Expected response:

```text
404 Not Found
```

Hal tersebut digunakan untuk memastikan resource benar-benar sudah tidak ditemukan setelah proses delete.

## Data-Driven Testing

Data-Driven Testing menggunakan file:

```text
data_test.csv
```

CSV terdiri dari minimal **3 skenario**:

1. Valid
2. Invalid
3. Edge Case

Contoh struktur CSV:

```csv
title,description
CSV Valid Test 001,Valid data from CSV 001
,Invalid test empty title
Edge case one character title,Edge
```

Data CSV digunakan oleh Postman Collection untuk menjalankan pengujian dengan data yang berbeda pada setiap iteration.

Dengan 3 baris data pengujian, collection dapat dijalankan sebanyak 3 iteration.

## Invalid Scenario

Request:

```text
09 - POST Invalid
```

digunakan untuk menguji data yang tidak memenuhi validasi API.

Expected response:

```text
400 Bad Request
```

Assertion memastikan API menolak data yang tidak valid.

## Edge Case Scenario

Request:

```text
10 - POST Edge Case
```

digunakan untuk menguji data pada kondisi batas atau edge case.

Pengujian memastikan API tetap memberikan response yang sesuai terhadap data pada kondisi tersebut.

## CSV Data Drive Request

Request:

```text
11 - POST CSV Data Drive
```

digunakan untuk menjalankan pengujian berdasarkan data dari:

```text
data_test.csv
```

Assertion pada request ini memvalidasi:

- Status code sesuai skenario
- Response berbentuk JSON
- Response time

## Run Locally

Pastikan **Postman CLI** sudah terinstall.

Collection dapat dijalankan dari terminal dengan:

```bash
postman collection run "Final Assignment AfterOffice.postman_collection.json" \
  -d "data_test.csv"
```

Perintah tersebut akan:

1. Membaca Postman Collection
2. Membaca file CSV
3. Menjalankan request secara otomatis
4. Menjalankan assertion
5. Menampilkan hasil testing pada terminal

## GitHub Repository Structure

Struktur repository:

```text
final-assignment-api-testing/
│
├── .github/
│   └── workflows/
│       └── postman-tests.yml
│
├── Final Assignment AfterOffice.postman_collection.json
├── data_test.csv
└── README.md
```

## GitHub Actions / CI-CD

Automated API testing diintegrasikan dengan **GitHub Actions**.

Workflow berada pada:

```text
.github/workflows/postman-tests.yml
```

Workflow otomatis berjalan ketika terjadi:

```text
Push → main
```

atau:

```text
Pull Request → main
```

Workflow menjalankan:

```text
Checkout Repository
        ↓
Install Postman CLI
        ↓
Run Postman Collection
        ↓
Load data_test.csv
        ↓
Execute API Tests
        ↓
Validate Assertions
```

Contoh workflow:

```yaml
name: Automated API Tests using Postman CLI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  automated-api-tests:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Install Postman CLI
        run: |
          curl -o- "https://dl-cli.pstmn.io/install/linux64.sh" | sh

      - name: Run Postman Collection
        run: |
          postman collection run "Final Assignment AfterOffice.postman_collection.json" \
            -d "data_test.csv"
```

## CI/CD Gatekeeper Testing

Pipeline juga diuji sebagai **gatekeeper**.

Untuk membuktikan pipeline dapat mendeteksi kegagalan, dilakukan intentional failure menggunakan step:

```yaml
- name: Gatekeeper Test - Intentional Failure
  run: exit 1
```

Ketika step tersebut dijalankan, GitHub Actions menghasilkan:

```text
Failure
Process completed with exit code 1
```

Hal tersebut membuktikan bahwa pipeline dapat memberikan status gagal ketika proses pengujian tidak memenuhi kondisi yang ditentukan.

Setelah bukti failure diperoleh, intentional failure dihapus dari workflow.

Pipeline kemudian dijalankan kembali hingga mendapatkan status:

```text
Success
```

Dengan demikian terdapat dua kondisi pengujian pipeline:

```text
Intentional Failure → ❌ Failure
Normal API Testing  → ✅ Success
```

## Test Result

Pada pengujian normal, collection berhasil menjalankan alur:

```text
Login
  ↓
GET All Labs
  ↓
POST Create Lab
  ↓
GET Lab by ID
  ↓
PUT Update Lab
  ↓
GET After Update
  ↓
DELETE Lab
  ↓
GET After Delete
  ↓
POST Invalid
  ↓
POST Edge Case
  ↓
POST CSV Data Drive
```

Data-driven testing menggunakan 3 skenario CSV sehingga collection dijalankan dalam beberapa iteration.

Pipeline GitHub Actions berhasil menjalankan automated API testing melalui Postman CLI.

Final pipeline:

```text
Status: Success
Failed Assertions: 0
```

## Evidence

Bukti pengujian yang digunakan pada assignment:

- Screenshot Postman Collection Runner
- Screenshot hasil assertion Postman
- Screenshot GitHub Actions Success
- Screenshot GitHub Actions Intentional Failure / Gatekeeper
- Log Postman CLI pada GitHub Actions

## Conclusion

Final Assignment ini mengimplementasikan automated API testing dengan cakupan:

- Authentication
- Automatic Token Handling
- CRUD API Testing
- Dynamic Lab ID
- Status Code Assertion
- Response Body Assertion
- Response Time Assertion
- Invalid Testing
- Edge Case Testing
- CSV Data-Driven Testing
- Postman CLI
- GitHub Actions
- CI/CD
- Gatekeeper Testing

Integrasi Postman CLI dan GitHub Actions memungkinkan API test dijalankan secara otomatis ketika terdapat perubahan pada branch `main` maupun Pull Request ke branch `main`.

Pipeline juga telah diuji menggunakan intentional failure untuk membuktikan bahwa CI/CD dapat berfungsi sebagai gatekeeper terhadap proses pengujian yang gagal.
