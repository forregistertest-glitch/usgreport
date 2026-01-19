================================================================================
PROJECT ANALYSIS: USG Reporting System (LAN-Based)
================================================================================

1. สรุปภาพรวมเทคโนโลยี (Technology Stack)
--------------------------------------------------------------------------------
ระบบถูกพัฒนาด้วยเทคโนโลยีพื้นฐานของเว็บ (Web Standards) เน้นการทำงานฝั่ง Client
ทำให้สามารถนำไปวางบน Shared Folder หรือ Web Server ภายในวง LAN ได้ทันที

1.1 Core Technologies:
    - HTML5: โครงสร้างหลักของหน้าฟอร์ม (Request, Report, Print View)
    - CSS3 (@media print): หัวใจสำคัญของการจัดหน้ากระดาษ A4
    - JavaScript (ES6+): Logic การจัดการข้อมูล คำนวณค่า และรับส่งข้อมูลระหว่างหน้า

1.2 Data Storage & Transfer (Techniques):
    - URLSearchParams: ใช้ส่งข้อมูลจากหน้า Request ไปยังหน้า Report (ผ่าน GET Method)
    - LocalStorage: ใช้ส่งข้อมูลขนาดใหญ่ (JSON Object) จากหน้า Report ไปยังหน้า Print View
    - Base64 Encoding: ใช้สำรองข้อมูลใน URL กรณี LocalStorage มีปัญหา

1.3 External Libraries & Assets:
    - html2pdf.bundle.min.js: ไลบรารีสำหรับแปลง HTML เป็น PDF (ทางเลือกเสริม)
    - Local Fonts (Sarabun): ใช้ไฟล์ .ttf ในเครื่อง (Offline) เพื่อความถูกต้องของฟอนต์ภาษาไทยและไม่ต้องต่อเน็ต

================================================================================

2. เทคนิคการพัฒนาที่โดดเด่น (Key Techniques Used)
--------------------------------------------------------------------------------

2.1 The Table Wrapper Technique (สำหรับ Print Layout):
    นี่คือเทคนิคสำคัญที่ใช้แก้ปัญหา Header/Footer ทับเนื้อหาเมื่อมีหลายหน้า
    - Concept: ใช้โครงสร้าง <table> หุ้มเนื้อหาทั้งหมด
    - <thead>: ใส่ div เปล่า (.page-header-space) เพื่อกันที่ด้านบนทุกหน้า
    - <tfoot>: ใส่ div เปล่า (.page-footer-space) เพื่อกันที่ด้านล่างทุกหน้า
    - ผลลัพธ์: Browser จะเว้นที่ว่างให้ Header/Footer (ที่เป็น position: fixed) ลอยอยู่ได้โดยไม่ทับเนื้อหา

2.2 Client-Side Data Persistence (การส่งข้อมูลข้ามหน้า):
    - Request.html -> Report.html:
        ใช้การเก็บค่าจาก Form -> สร้าง Query String -> Redirect
    - Report.html -> Print_view.html:
        ใช้การ Pack ข้อมูลทั้งหมดเป็น JSON Object -> บันทึกลง LocalStorage ('printData') -> เปิดหน้าใหม่ -> หน้าใหม่ดึงข้อมูลมา Render

2.3 Auto-Logic & Automation:
    - Auto-Check Examined: หากมีการสั่งตรวจ (Request) อวัยวะใด ระบบจะติ๊ก 'Examined' ในหน้า Report ให้ทันที
    - Dynamic Text Generation: สร้าง Text Report โดยคำนวณความยาวเส้นขีด (Dash) ให้สัมพันธ์กับความยาวหัวข้ออัตโนมัติ
    - HL7 Generation: สร้างไฟล์ text ตามมาตรฐานการแพทย์ (.hl7) เพื่อนำเข้าสู่ระบบ HIS ได้

================================================================================

3. การกำหนดค่าและการตั้งค่า (Configuration & Settings)
--------------------------------------------------------------------------------

3.1 Page Setup (CSS @media print):
    - Paper Size: A4 Portrait
    - Browser Margin: ตั้งค่าเป็น 10mm (บน/ล่าง) เพื่อให้ Table Wrapper ทำงานต่อได้
    - Content Margin: Top 25mm / Bottom 20mm (ผ่าน Spacer ในตาราง)

3.2 Spacer Dimensions (พื้นที่กันชน):
    - Header Space (.page-header-space): สูง 25px
    - Footer Space (.page-footer-space): สูง 35px
    - Body Padding: ลบ Padding ที่ซ้ำซ้อนออก เพื่อป้องกันพื้นที่ว่างเกินจำเป็นในหน้าสุดท้าย

3.3 Data Flow Constraints:
    - Date Format: จัดรูปแบบวันที่ (DD/MM/YYYY HH:mm) ที่ฝั่ง JavaScript ก่อนส่งไปแสดงผล เพื่อแก้ปัญหา NaN
    - Follow-up Logic: ตรวจจับคำว่า "Follow up" ในประวัติ เพื่อติ๊ก Checkbox อัตโนมัติ

================================================================================

4. SYSTEM DIAGRAM & FUNCTION FLOW
--------------------------------------------------------------------------------

[User/Admin] 
     |
     v
(1) REQUEST FORM (request.html)
     | - Input: Patient Info, Doctor, Priority
     | - Action: Select Organs (Checkbox)
     | - Logic: Validate HN
     |
     | (Data Transfer: URL Parameters ?hn=...&organs=...)
     v
(2) REPORTING INTERFACE (report.html)
     | - Auto-Load: Data from URL
     | - Logic: Auto-check 'Examined' based on Order
     | - Input: Fill Findings / Measurements / Select Templates
     | - Action Buttons:
     |      |-- [View Text Report] -> Popup Text Window (Copy Paste purpose)
     |      |-- [Export HL7] ------> Download .hl7 file (Integration)
     |      |-- [Print Preview] ---> (Save JSON to LocalStorage) -> Open Window
     v
(3) PRINT VIEW (print_view.html)
     | - Load: JSON from LocalStorage
     | - Render: Map Data to HTML Elements
     | - Layout: Apply "Table Wrapper" Structure
     |      |-- Fixed Header (Date)
     |      |-- Fixed Footer (HN, Pet, Date)
     |      |-- Content Flow (Findings)
     |
     v
[PRINTER / PDF] (Final A4 Output)

================================================================================
