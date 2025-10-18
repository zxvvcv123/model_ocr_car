# 🚗 YOLOModelTest

ระบบจำลอง Parking Watch สำหรับตรวจจับรถที่จอดอยู่ในพื้นที่นานเกินกำหนด
โดยใช้ YOLOv8 ในการตรวจจับป้ายทะเบียน และ OCR (EasyOCR) สำหรับอ่านข้อความบนป้าย

Notebook เดียว (yolomodeltest.ipynb) ครอบคลุมทุกขั้นตอน:

## 📂 การเตรียมข้อมูลและแบ่ง Dataset

เนื่องจาก Dataset ใหญ่เกินทีจะ Upload ลง Git ได้  
จึงขออนุญาตให้ [ดาวน์โหลด](https://drive.google.com/drive/folders/1qC0drXGKsUKW8W3nQDzFIzZLcMBLIuOF?usp=sharing) ผ่าน Google Drive แทน

## 🧠 การเทรนโมเดล YOLO

## 🎥 การจำลองระบบ Parking Watch

🧭 โครงสร้างไฟล์
yolomodeltest/
│  
├── yolomodeltest.ipynb      ← Notebook หลัก (รวมทุกส่วน)  
│  
├── data/  
│   ├── images_all/          ← ใส่ภาพทั้งหมดที่มี  
│   └── labels_all/          ← ใส่ไฟล์ label YOLO (.txt)  
│  
├── datasets/  
│   └── YOLO/                ← ผลลัพธ์หลังแบ่ง train/val/test  
│       ├── images/  
│       └── labels/  
│  
├── runs/                    ← ที่เก็บผลการเทรน YOLO  
│   └── detect/  
│  
├── output_longstay/         ← เก็บภาพรถที่จอดเกินเวลา  
└── fast_alpr/               ← โมดูล OCR สำหรับอ่านป้ายทะเบียน  

## ⚙️ การติดตั้ง (ครั้งเดียวก่อนใช้งาน)

เปิด Jupyter Notebook แล้วรันคำสั่งด้านล่างนี้ใน Cell แรกของไฟล์ yolomodeltest.ipynb

```bash
pip install ultralytics opencv-python numpy easyocr tqdm
```

> 💡 แนะนำให้ใช้ Python ≥ 3.9 และมี GPU (CUDA) จะเทรนได้เร็วขึ้น

## ขั้นตอนการใช้งานใน Notebook

### 1. แบ่งชุดข้อมูล (Split Dataset)

- แบ่งภาพและ labels จาก data/images_all เป็น 3 ส่วน:

    | #          | Ratio |
    |:-----------|:-----:|
    | Train      | 70%   |
    | Validation | 20%   |
    | Test       | 10%   |

- ใส่ภาพทั้งหมดไว้ใน data/images_all/
- ใส่ label ของ YOLO (.txt) ใน data/labels_all/
- รัน Cell ในหัวข้อ SPLIT DATASET
- เมื่อเสร็จแล้วจะได้โฟลเดอร์:

    datasets/YOLO/images/train/  
    datasets/YOLO/images/val/  
    datasets/YOLO/images/test/  

- แสดงข้อความ: Dataset split complete!

### ขั้นตอนที่ 2: เทรนโมเดล YOLOv8

- ใช้โมเดล YOLOv8 สำหรับตรวจจับป้ายทะเบียนรถ
- สร้างไฟล์ datasets/YOLO/data.yaml ดังนี้:

    ```txt
    train: datasets/YOLO/images/train
    val: datasets/YOLO/images/val
    nc: 1
    names: ['license_plate']
    ```

- ใน Notebook รันส่วน TRAIN YOLO MODEL
ระบบจะเริ่มเทรนโมเดลและบันทึกผลไว้ที่ runs/detect/train/weights/best.pt
- หลังจบการเทรน จะได้ไฟล์น้ำหนัก (best.pt) สำหรับใช้งานจริง

### ขั้นตอนที่ 3: จำลองระบบ Parking Watch

- ระบบจะเปิดกล้อง (หรือไฟล์วิดีโอ) แล้วตรวจจับป้ายทะเบียน
หากเจอทะเบียนเดิมค้างไว้นานเกิน 120 วินาที → ถ่ายภาพเก็บใน output_longstay/
- แก้ path กล้องหรือวิดีโอในตัวแปร VIDEO_SOURCE
- VIDEO_SOURCE = 0  # 0 = กล้องเว็บแคม, หรือใส่ path ของไฟล์ .mp4
- รันส่วน PARKING WATCH SIMULATION

- ดู log จาก console:

    ```bash
    [10:32:11] Detected plate: 1กข1234
    [10:34:15] Plate 1กข1234 parked >120s → snapshot saved
    ```

ตรวจสอบภาพรถที่จอดนานใน output_longstay/

### Repository ที่เกี่ยวข้อง

- [OCR Model](https://github.com/zxvvcv123/model_ocr_car)
- [Detection Model](https://github.com/UnKnowConut/YOLO)
- [Notify Service](https://github.com/Harin3Bone/license-plate-detection-notify-service)