# Subscription Calendar

## แก้ CORS / โหลดไม่สำเร็จ

ถ้าเปิดไฟล์ `index.html` ตรงๆ (double-click) จะได้ **Origin null** และ n8n จะ block request (CORS)

**ให้รันผ่าน local server แทน:**

```bash
cd subscription-calendar
npx serve .
```

แล้วเปิดเบราว์เซอร์ที่ **http://localhost:3000**

หรือใช้ Python:

```bash
cd subscription-calendar
python3 -m http.server 8000
```

แล้วเปิด **http://localhost:8000**

---

ใน n8n ให้ใช้ **Production URL** (ไม่ใช่ Test URL) และ path ต้องตรงกับที่ใส่ในโค้ด: `https://n8n.willy.moda/webhook/my-subscription` (POST).

---

## คอลัมน์ใน Table (n8n)

| คอลัมน์ | ใช้แสดง |
|--------|--------|
| `name` (หรือ title, subscriptionName) | ชื่อ subscription |
| `date` / `paymentDate` / `dueDate` | วันที่จ่าย (รูปแบบ YYYY-MM-DD หรือ day 1-31) |
| `price` (หรือ amount) | ราคา สำหรับ Monthly spend |
| **`icon`** (หรือ iconSvg, icon_svg) | **SVG เป็นข้อความ** เช่น `<svg xmlns="..." viewBox="0 0 24 24"><path d="M13 2L11 13h4l-2 11"/></svg>` หรือ URL รูป |
| **`iconColor`** (หรือ icon_color, color) | **สีจุดมุม** เช่น `#22c55e` หรือ `green` |

ถ้ามีคอลัมน์ Icon จะแสดงเป็นกล่องมีไอคอน + จุดสีแบบในรูป

---

## แก้ปัญหา "โหลดแค่ 1 รายการทั้งที่ส่ง 2 อัน"

1. **n8n ส่งแค่ First Entry**  
   ในโหนด **Respond to Webhook** อย่าใช้ "First Entry JSON" ถ้าต้องการส่งหลายรายการ  
   ให้ส่งเป็น **array** ทั้งก้อน เช่น ใช้ "Respond With" แบบรวมทุก item เป็น array แล้วส่ง `{{ $json }}` หรือใช้โหนดรวม (Merge/Aggregate) แล้วตอบกลับเป็น array

2. **อีกรายการอยู่คนละเดือน**  
   ปฏิทินแสดงแค่ **เดือนปัจจุบัน**  
   ตัวอย่าง: Github (12 ก.พ.) แสดงในเดือนก.พ., N8N (31 ม.ค.) แสดงในเดือนม.ค.  
   ถ้าอยากเห็นทั้งคู่ต้องเปิดดูเดือนที่ตรงกับ dueDate ของแต่ละรายการ (หรือเพิ่มปุ่มเปลี่ยนเดือนในอนาคต) แทนข้อความชื่อ
