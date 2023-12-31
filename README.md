# Real-time Loan Fraud Detection from Streaming Card and Loan Data
โปรเจคนี้จะ detect ข้อมูลแบบ real-time จากสองที่มาคือ Card data และ Loan data ซึ่งจะทำการ generate 
ข้อมูลและส่งไปที่ topic ที่สร้างไว้ 2 topics คือ Card topic และ Loan topic โดยใช้บริการ global real-time messaging 
ของ google cloud platform และสร้าง subscription เพื่อรับข้อมูลที่ได้ generate ออกมา 2 topics คือ Card topic 
และ Loan topic เช่นกัน เพื่อทำการประมวลผลตาม Requirements ที่ได้กำหนดไว้เพื่อตรวจจับการฉ้อโกงเงินกู้และบันทึกลงใน Big query
เพื่อนำไปวิเคราะห์ข้อมูลต่อไป 

# ตัวอย่างข้อมูลทีทำการ Generate ขึ้น
ข้อมูลที่ Generate ทำการ Generate ขึ้นจาก 2 ที่ซึ่ง Generate จากไฟล์ Generate_loan_data.py คือ 
1. Card data เป็นข้อมูลต่างๆของ Card ซึ่งมีจำนวน 8 Columns คือ \
 1.1 Customer_id ซึ่งคือ unique customer identification ของเจ้าของบัตร \
 1.2 Relationship_no คือ Relationship number of Card \
 1.3 Card_type คือประเภทของบัตรซึ่งมี 8 ประเภทคือ 1.Issuers 2.AccountNow_Prepaid_Cards 3.MasterCard 4.Discover 5.Capital_One 6.American_Express 7.Network 
 8.Visa \
 1.4 Max_credit_limit คือ max spent limit ของ customer ใน 1 เดือน \
 1.5 Total_Spent คือ ค่าใช้จ่ายของ customer ใน 1 เดือน \
 1.6 Cash_withdrwn คือ จำนวนเงินที่ customer ถอน ใน 1 เดือนจากตู้ ATM ซึ่งรวมอยู่ใน Total_Spent เรียบร้อย \
 1.7 Cleared_amount คือ การชำระค่าใช้จ่ายของ customer ใน 1 เดือน \
 1.8 Last_date คือ วันสุดท้ายของเดือนและเป็นวัยสุดท้ายในการชำระเงินนั่นเอง โดยกำหนดทุกปีเป็น 2019   
2. Loan data เป็นข้อมูลเงินกู้ของ customer ต่างๆซึ่งมี 7 Columns คือ \
 2.1 Customer_id คือ unique identification ของเจ้าของบัตร \
 2.2 Customer_category คือ หมวดหมู่ของ customer ซึ่งมี 5 ประเภทคือ 1.Serviceman 2.Businessman 3.Student 4.Others 5.Self \
 2.3 Loan_id คือ  unique loan identification ของ customer นั้นๆ \
 2.4 Loan_category คือ หมวดหมู่ของเงินกู้ ซึ่งมี 3 ประเภทคือ 1.Medical_Loan 2.Personal_Loan 3.Education_Loan \
 2.5 Due_date คือ วันที่ครบกำหนดการผ่อนชำระสำหรับเดือนนั้นๆ โดยกำหนดทุกปีเป็น 2019 
 2.6 Due_amount คือ จำนวนเงินค่างวดที่ครบกำหนด \
 2.7 Payment_date คือ วันที่ชำระเงินจริง โดยกำหนดทุกปีเป็น 2019 

# Requirements ที่กำหนดในการตรวจสอบเงินกู้ว่าเป็นการฉ้อโกงหรือไม่
1. Credit card defaulter requirements:\
1.1 กำหนดให้ customer ได้รับ 1 คะแนนในกรณีที่ชำระเงินไม่ครบตามที่กำหนด ซึ่งหมายความว่าลูกค้าไม่สามารถชำระเงินได้ตั้งแต่ 80% ของยอดการใช้จ่ายรายเดือนของเขาขึ้นไป \
1.2 กำหนดให้ customer ได้รับ 1 คะแนนในกรณีที่ลูกค้าใช้จ่ายจำนวนเต็มของวงเงินสูงสุดของเขา แต่ไม่ชำระเต็มจำนวนที่ครบกำหนด \
1.3 หาก customer ตรงตามทั้งสองเงื่อนไขด้านบนในเดือนใดๆ กำหนดให้ลูกค้าได้รับ 1 คะแนนเพิ่มเติม \
2. Loan defaulter requirements:\
2.1 สำหรับประเภทสินเชื่อบุคคล (personal loan) ธนาคารไม่ยอมรับการชำระเงินไม่ครบตามกำหนดหรือชำระเกินกำหนดเวลา \
2.2 ถ้า customer ไม่ได้ชำระเงินผิดกำหนดทั้งหมด 4 งวดขึ้นไป หรือไม่ได้ชำระเงินผิดกำหนดติดต่อกัน 2 งวด จะถือว่าเป็นผู้กู้ที่ไม่ชำระเงินสินเชื่อส่วนบุคคลตามกำหนด 

# การเตรียมการในทำโปรเจค
1. สร้าง Topics ขึ้นมา 2 Topics คือ Card topic และ Loan topic โดยใช้บริการ pub/sub ของ google cloud platform  
<img src = 'image/8.PNG'>

2. สร้าง subcription ของ Card topic และ Loan topic ขึ้นมา 
<img src = 'image/9.PNG'>

3. uploadfile Generate_loan_data.py และ Detect_loan.py บน Cloud Shell Editor 
4. ทำการสร้าง Bigquery เพื่อรับข้อมูลโดย Dataset ต้องมี Region เดียวกับ cloud storage ในขั้นตอนถัดไป 
และมีจำนวน 4 Columns ดังนี้ 
<img src = 'image/21.PNG'>

 Column ที่ 1 คือ Customer_id สำหรับบันทึก id ลูกค้า \
 Column ที่ 2 คือ Card_Defaulter_Fraud_points สำหรับบันทึกคะแนน Fraud_points ของ customer_id ที่ตรง requiements ของ Credit Card defaulter \
 Column ที่ 3 คือ Personal_Loan_Defaulter_Consecutive_Missing สำหรับบันทึกคะแนน Consecutive_Missing ซึ่งคือ \
 จำนวนครั้งที่ขาดการชำระเงินติดต่อกันนับเป็น 1 เดือน 1 ครั้งตาม Loan defaulter requirements \
 Column ที่ 4 คือ Both_Defaulters ซึ่งจะเป็น yes เมื่อ 2 เงื่อนไขตรง requirements ของทั้ง Credit Card defaulter และ Loan defaulter ถ้าไม่ตรงทั้งสองเงื่อนไขจะเป็น No\
 schema ของ table
<div align="left"> 
<pre><code>
[
 {
  "mode": "NULLABLE",
  "name": "Customer_id",
  "type": "STRING"
 },
 {
  "mode": "NULLABLE",
  "name": "Card_Defaulter_Fraud_points",
  "type": "STRING"
 },
 {
  "mode": "NULLABLE",
  "name": "Personal_Loan_Defaulter_Consecutive_Missing",
  "type": "STRING"
 },
 {
  "mode": "NULLABLE",
  "name": "Both_Defaulters",
  "type": "STRING"
 }
]
   </code></pre>
</div>

5.สร้าง cloud storage สำหรับเก็บข้อมูลและ folder temp \
Example.
<img src = 'image/6.PNG'>

6. Enable Dataflow API 
<img src = 'image/27.PNG'>

# เริ่มต้นโปรเจค
1. ทำการ run Generate_loan_data.py บน Cloud Shell Editor 
<div align="center"> 
  <pre><code>python Generate_loan_data.py</code></pre>
</div>

จะได้ตัวอย่างข้อมูลคือที่ generateขึ้นมาดังนี้  
1.1 ตัวอย่างข้อมูลที่ generate ไป Card topic
<img src = 'image/11.PNG'>

1.2 ตัวอย่างข้อมูลที่ generate ไป Loan topic
<img src = 'image/10.PNG'>

2.สร้าง Dataflow ขึ้นมาโดยใช้คำสั่งบน cloud shell
<div align="left"> 
  <pre><code>
   python Detect_loan.py \
  --project= (your projectid) \
  --region=(your region) \
  --input_subscription_card=projects/(your projectid)/subscriptions/Card\
  --input_subscription_loan=projects/(your projectid)/subscriptions/Loan\
  --output_table=(your projectid).(your dataset).(your table) \
  --runner=DataflowRunner \
  --temp_location=gs://(your cloud storage)/temp \
  --num_workers=1 \
  --max_num_workers=1 \
  --experiment=use_unsupported_python_version
  </code></pre>
</div>
จะได้ Dataflow pipeline ขึ้นมาดังนี้ 

<img src = 'image/22.PNG'>
<img src = 'image/23.PNG'>
<img src = 'image/24.PNG'>
<img src = 'image/25.PNG'>

3.รอสักพักใหญ่ๆ เพื่อประมวลผล \
4. ตัวอย่างข้อมูลที่บันทึกลงไปใน Bigquery
<img src = 'image/26.PNG'>



