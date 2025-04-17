# คู่มือการใช้งาน Docker Compose โครงสร้างแบบ Extends

## โครงสร้างโปรเจค

```
├── .gitignore                        # ไฟล์กำหนดให้ Git ไม่สนใจไฟล์บางประเภท
├── .dockerignore                     # ไฟล์กำหนดให้ Docker ไม่สนใจไฟล์บางประเภท
├── docker-compose.yml                # ไฟล์ docker-compose หลัก
│
├── compose-extends/                  # โฟลเดอร์เก็บไฟล์ compose ย่อย
│   ├── dotnet-api-1.yml              # ไฟล์ compose สำหรับ API 1
│   └── dotnet-api-2.yml              # ไฟล์ compose สำหรับ API 2
│
├── dotnet-api-1/                     # โฟลเดอร์โปรเจค API 1
│   ├── .env                          # ไฟล์ env สำหรับ API 1
│   ├── Dockerfile                    # Dockerfile สำหรับ API 1
│   ├── publish/                      # โฟลเดอร์เก็บไฟล์ publish
│   ├── cert/                         # โฟลเดอร์เก็บ certificate
│   └── logs/                         # โฟลเดอร์เก็บ log
│
└── dotnet-api-2/                     # โฟลเดอร์โปรเจค API 2
    ├── .env                          # ไฟล์ env สำหรับ API 2
    ├── Dockerfile                    # Dockerfile สำหรับ API 2
    ├── publish/                      # โฟลเดอร์เก็บไฟล์ publish
    ├── cert/                         # โฟลเดอร์เก็บ certificate
    └── logs/                         # โฟลเดอร์เก็บ log
```

## วิธีการใช้งาน

### 1. เตรียมไฟล์ Publish

ทำการ publish แอปพลิเคชันของแต่ละ API:

```bash
# สำหรับ API 1
cd path/to/api1-source
dotnet publish -c Release -o ../docker-project/dotnet-api-1/publish

# สำหรับ API 2
cd path/to/api2-source
dotnet publish -c Release -o ../docker-project/dotnet-api-2/publish
```

### 2. การเริ่มใช้งาน

**เปิดใช้งานทั้งหมด**:
```bash
docker-compose up -d
```

**เปิดใช้งานเฉพาะ service ที่ต้องการ**:
```bash
docker-compose up -d dotnet-api-1
```

**หยุดการทำงานทั้งหมด**:
```bash
docker-compose down
```

### 3. การปรับแต่งค่า

แต่ละ service จะมีไฟล์ .env แยกกัน ทำให้คุณสามารถปรับแต่งค่าได้อย่างอิสระ:

**dotnet-api-1.env**:
| ตัวแปร | คำอธิบาย | ค่าเริ่มต้น |
|--------|---------|------------|
| DOTNET_VERSION | เวอร์ชัน .NET Core | 9.0 |
| APP_DLL | ชื่อไฟล์ dll ของแอป | YourApiProject.dll |
| API_CONTAINER_NAME | ชื่อคอนเทนเนอร์ | dotnet-api-1 |
| HTTP_PORT | พอร์ต HTTP | 8080 |
| HTTPS_PORT | พอร์ต HTTPS | 8443 |

**dotnet-api-2.env**:
| ตัวแปร | คำอธิบาย | ค่าเริ่มต้น |
|--------|---------|------------|
| DOTNET_VERSION | เวอร์ชัน .NET Core | 8.0 |
| APP_DLL | ชื่อไฟล์ dll ของแอป | Api2Project.dll |
| API_CONTAINER_NAME | ชื่อคอนเทนเนอร์ | dotnet-api-2 |
| HTTP_PORT | พอร์ต HTTP | 9080 |
| HTTPS_PORT | พอร์ต HTTPS | 9443 |

## การเพิ่ม Service ใหม่

1. สร้างไฟล์ใน `compose-extends/` เช่น `new-service.yml`
2. สร้างโฟลเดอร์สำหรับ service ใหม่ เช่น `new-service/`
3. สร้างไฟล์ `.env` ในโฟลเดอร์ service ใหม่
4. เพิ่ม service ในไฟล์ `docker-compose.yml` หลัก:

```yaml
new-service:
  extends:
    file: ./compose-extends/new-service.yml
    service: new-service
  env_file:
    - ./new-service/.env
  networks:
    - backend-network
```

## ข้อดีของโครงสร้างแบบนี้

1. **แยกการกำหนดค่า**: แต่ละ service มีไฟล์ config แยกกัน ทำให้จัดการง่าย
2. **รองรับหลาย .NET version**: สามารถรันแอปพลิเคชันบน .NET version ที่ต่างกันได้
3. **ยืดหยุ่นในการ deploy**: สามารถเปิดใช้งานเฉพาะ service ที่ต้องการได้
4. **แยกไฟล์ config ตามหน้าที่**: ทำให้ง่ายต่อการบำรุงรักษาและขยายระบบ
5. **การจัดการ Git ที่เหมาะสม**: ไม่เพิ่มไฟล์ที่ไม่จำเป็นเข้าสู่ระบบ version control