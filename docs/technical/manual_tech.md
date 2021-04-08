
# Technical
---
## โครงสร้างการเขียน Code สำหรับระบบอื่น {#Heading8}
การเรียก Service Docsign Server ระบบอื่นๆ สามารถเรียกใช้ได้ โดยถอดตัวอย่างจากระบบ PABI2 (Python2)

1. การเชื่อมต่อเข้าระบบ โดยถ้าเชื่อมต่อสำเร็จ ระบบจะได้รับค่า uid กลับมา

```python
    import xmlrpclib

    # connect with docsign
    models = xmlrpclib.ServerProxy('{}/xmlrpc/common'.format(url))
    uid = models.authenticate(db, username, password, {})
```


**หมายเหตุ** url, db, username, password มาจากการตั้งค่าจากหัวข้อที่ 2

2. หลังจากเชื่อมต่อเข้าระบบแล้ว สามารถที่จะเรียกใช้ function ที่มีอยู่ใน Docsign Server โดยจะส่งข้อมูลเอกสารไปยัง Docsign Server เพื่อทำการสร้างเอกสารและลงลายมือชื่ออิเล็กทรอนิกส์ ซึ่งค่าที่ Server จะส่งกลับมาจะอยู่ในรูปแบบ dictionalry of list ดังนี้

```PYTHON
    [
        {
            'id': 11293, 
            'name': 'DV19000456',
            'link_download': '<url_server>/web/content/<id>/<filename>?download=true', 
            'status': 'OK', 
            'errorMessage': False
        }
    ]
```

```PYTHON
    # call method in server
    models = xmlrpclib.ServerProxy('{}/xmlrpc/object'.format(url))
```

```PYTHON
    res_ids = models.execute_kw(
        db, uid, password, 'account.printing','action_call_service', [voucher_dict]
        )
```

การเรียกใช้ Function ภายใน execute_kw ให้กำหนดข้อมูลดังนี้

- uid มาจากการที่ระบบ return มาจากการเชื่อมต่อระบบ
- 'account.printing' คือ model ใน docsign server ที่ใช้ในการเก็บเอกสารทั้งหมด
- 'action_call_service' คือ Function ที่อยู่ภายใน model account.printing ใน Server
- [voucher_dict] คือ ข้อมูลที่จะส่งไปในรูปแบบ dictionary (ตัวอย่างอยู่ในเอกสารเพิ่มเติม)

3. รับค่าที่ระบบส่งกลับมา และตรวจสอบจาก status
หากเป็น OK จะมีค่า link_download ให้นำ link ที่ได้ไปเก็บไว้ในเอกสารนั้นๆ เมื่อ user ต้องการดูเอกสาร ก็จะสามารถคลิกไปที่ link นั้น ๆ เพื่อ download เอกสารมาเปิดดูได้
ถ้า status ไม่ใช่ OK ระบบจะแสดง errorMessage และ link_download จะเป็น False สามารถนำค่าตรงส่วนนี้ไปแสดงเป็น Error ในระบบได้



## เอกสารเพิ่มเติม {#Heading9}
การเรียกใช้งานฟอร์ม Delivery Order / Tax Invoice จาก mySale สามารถทำได้โดยการเรียก docform เป็น delivery_order_tax_receipt (ในระบบ PABI จะไม่มีการเรียก Form นี้)

### ตารางโครงสร้างข้อมูลหัวข้อเอกสารที่ใช้สำหรับการลงมือชื่ออิเล็กทรอนิกส์ {#Heading9_1}

{{ read_csv('technical/csv/table1.csv') }}{: .center-image}

### ตารางโครงสร้างข้อมูลสินค้าในเอกสารที่ใช้สำหรับการลงมือชื่ออิเล็กทรอนิกส์ (printing_lines) {#Heading9_2}



{{ read_csv('technical/csv/table2.csv') }}{: .md-typeset__table}

### ตารางโครงสร้างข้อมูลสินค้าในเอกสารที่ใช้สำหรับการลงมือชื่ออิเล็กทรอนิกส์ (payment_diff_lines) {#Heading9_3}

{{ read_csv('technical/csv/table3.csv') }}{: .center-image}

    <!-- :file: csv/table3.csv
    :widths: 10,20,50,20,20,10
    :header-rows: 1 -->


### ตัวอย่างการส่งข้อมูล invoice ไปที่ระบบ {#Heading9_4}


```PYTHON
    {
        # header
        'doctype': 'T03',
        'docform': 'customer_tax_receipt',
        'lang_form': 'th',
        'number': 'RC2563/08-0001',
        'cancel_form': True,
        'customer_code': '001203',
        'customer_name': 'นายทดสอบ ระบบ',
        'seller_name': 'สำนักงาน',
        'currency': 'THB',
        'date_document': '2020-08-01',
        'create_document': '2020-08-01',
        'operating_unit': 'สก.',
        'purpose_code': 'TIVC99',                              # ใช้สำหรับกรณีออกใหม่ทดแทน
        'purpose_reason_other': 'เปลี่ยนสาขาผู้ขาย',               # ใช้สำหรับกรณีออกใหม่ทดแทน
        'notes': '',
        'state_draft': False,
        # customer information
        'customer_street': 'อาคารตึกใหญ่ 12 ซอยนราธิวาส',
        'customer_street2': '',
        'customer_city': '',
        'customer_state': '',
        'customer_country': '',
        'customer_subdistrict': '',
        'customer_building_number': '111',
        'customer_zip': '10110',
        'customer_country_code': 'TH',
        'customer_province_code': '10',
        'customer_district_code': '1001',
        'customer_subdistrict_code': '100103',
        'customer_vat': voucher.partner_id.vat,
        'customer_phone': '0829330432',
        'customer_email': 'test@email.com',  
        'customer_taxbranch_code': '00000',
        'customer_taxbranch_name': 'สำนักงานใหญ่',
        'customer_is_company': True,
        # seller information
        'seller_street': 'ตึกใหม่ 22/11',
        'seller_street2': '',
        'seller_city': '',
        'seller_state': '',
        'seller_country': '',
        'seller_subdistrict': '',
        'seller_building_number': '111',
        'seller_zip': '10300',
        'seller_country_code': 'TH',
        'seller_province_code': '10',
        'seller_district_code': '1001',
        'seller_subdistrict_code': '100103',
        'seller_vat': '0092839405932',
        'seller_phone': '089-222-1111',
        'seller_fax': '0293940506',
        'seller_email': 'email@email.com',
        'seller_taxbranch_code': '00000',
        'seller_taxbranch_name': 'สำนักงานใหญ่',
        'amount_untaxed': 93.2,
        'amount_tax': 3,
        'amount_total': 96.2,
        # origin
        'origin_id': 'RT2564/01-0004'
        'system_origin_name': 'pabi2',
        'system_origin_number': 'RC200003021',
        'user_sign': 'Admin',
        'validate_by': 'Mr.Adam',
        'validate_sign': '/9j/4AAQSkZJRgABAQEA...',
        'approved_by': 'Admin',                               # TODO: Waiting new pg.
        # payment method
        'payment_method': voucher.receipt_type,
        'check_no': '0011230412',
        'check_date': '2020-08-01',
        'bank_name': 'BBL',
        'bank_branch': 'สีลม',
        'rdx_no': '',                                         # TODO: Waiting RDX
        # lines
        'printing_lines': [(0, 0, {
        'name': 'คำอธิบาย',
        'quantity': 3,
        'price_unit': 100.0,
        'price_subtotal': 300,
        'product_code': '10020',
        'product_name': 'กล่องกระดาษ',
        'taxes': 'Output Vat 7%',
        'taxes_percent': 7,
    })]
        'payment_diff_lines': [(0, 0, {
        'note': 'คำอธิบาย',
        'amount': 3
    })]
    }
```

### Python Package เพิ่มเติม {#Heading9_5}

การติดตั้งโมดูล docsign_template_form จะต้องมีการติดตั้ง Package python เพิ่มเติมดังนี้

```python
    pip3 install pypng
    pip3 install pyqrcode
    pip3 install python-barcode
```

### Github Reference {#Heading9_6}
| ref: 'https://github.com/pabi2/pb2_addons/pull/1945'
| ref: 'https://github.com/pabi2/pb2_addons/pull/1916'
| ref: 'https://github.com/pabi2/pb2_addons/pull/1906'
