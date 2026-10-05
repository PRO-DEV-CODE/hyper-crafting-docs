# การเชื่อมต่อ

`Hyper_Crafting` **ไม่มี export เป็นของตัวเอง** และไม่มี resource ไหนในเซิร์ฟเวอร์เรียกใช้มัน — มันเป็นปลายทางที่ผู้เล่นเข้าถึงผ่านโต๊ะในแผนที่เท่านั้น

หน้านี้จึงรวบรวม **export ของ resource อื่นที่ระบบนี้เรียกใช้** เพื่อให้รู้ว่าต้องมีอะไรอยู่ในเซิร์ฟเวอร์บ้าง และถ้าจะย้าย/เปลี่ยน resource ตัวไหนจะกระทบตรงไหน

## ถ้าจะทำแบบนี้ ใช้ตัวนี้

| อยากทำอะไร | ใช้ export | ฝั่ง | ที่ใช้ในระบบนี้ |
|---|---|---|---|
| ให้ของในโต๊ะมีสีความหายากเหมือนในกระเป๋า | `Hyper_Inventory:GetRarityInfo` | client | สีขอบ/พื้นการ์ดไอเทมทุกใบ |
| ดึงคำอธิบายไอเทมมาโชว์ตอน hover | `Hyper_Inventory:GetItemDescription` | client | ข้อความในป้ายช่วยเหลือ |
| ดึงการ์ดรายละเอียดเต็ม (คำเตือน หัวข้อ ค่าสถานะ) | `Hyper_Inventory:GetItemDetail` | client | บล็อกล่างของป้ายช่วยเหลือ |
| บอกว่าไอเทมอยู่หมวดอะไร | `Hyper_Inventory:GetItemCategoryInfo` | client | บรรทัด "ประเภท" ในป้ายช่วยเหลือ |
| อ่านว่าผู้เล่นมีตู้อะไรบ้าง | `Hyper_Vault:GetPersonalVaults` | server | รายการในดรอปดาวน์เลือกตู้ |
| นับของในตู้ | `Hyper_Vault:VaultGetCounts` | server | ตัวเลข "มีอยู่" ที่รวมกระเป๋ากับตู้ |
| หักของจากตู้ | `Hyper_Vault:VaultTakeItems` | server | ตอนหักวัตถุดิบที่กระเป๋าไม่พอ |
| คืนของเข้าตู้ | `Hyper_Vault:VaultAddItems` | server | ตอนยกเลิกคราฟหรือคืนของ |
| แจ้งเตือนผู้เล่น | `Hyper_Notifyall:sendNotify` | client | ผลคราฟ ข้อความผิดพลาด |
| ป้ายช่วยเหลือตอนเข้าใกล้โต๊ะ | `Hyper_Notifyall:showHelpNotify` / `hideHelpNotify` | client | "กด E เปิดโต๊ะคราฟ" |
| ป้ายข้อความบนจอ | `Hyper_Notifyall:ShowTextUI` | client | ข้อความกำกับโต๊ะ |
| เก็บ audit trail | `Hyper_Discordlogs:CreateLog` | server | ทุกครั้งที่ของหรือเงินเปลี่ยนมือ |

## resource ที่ต้องมี

| Resource | จำเป็นแค่ไหน | ถ้าไม่มีจะเป็นยังไง |
|---|---|---|
| `Hyper_Notifyall` | **บังคับ** (ประกาศใน `fxmanifest`) | แจ้งเตือนและ Text UI ไม่ขึ้น |
| `Hyper_Inventory` | เกือบบังคับ | รูปไอเทมหาย การ์ดเป็นสีเทาหมด ป้ายช่วยเหลือว่าง |
| `Hyper_Vault` | ไม่บังคับ | ปิด `Settings.useVault` แล้วใช้แค่ของในกระเป๋าได้ |
| `Hyper_Discordlogs` | ไม่บังคับ | ไม่มี audit trail (เว็บฮุกตรงยังทำงาน) |
| `Hyper_Check` | ไม่บังคับ | ข้าม shared script ตัวนี้ได้ |

ทุก export ถูกเรียกผ่านตัวดักข้อผิดพลาด ถ้า resource ปลายทางไม่อยู่หรือยังไม่สตาร์ท ระบบจะใช้ค่าสำรองแทนแล้วทำงานต่อ ไม่พังทั้งโต๊ะ

## Event ภายใน

Event เหล่านี้เป็นของระบบเอง ใช้คุยกันระหว่างไคลเอนต์กับเซิร์ฟเวอร์ ไม่ได้ตั้งใจให้ resource อื่นเรียก — **อย่ายิงจากที่อื่น** เพราะด่านตรวจทั้งหมดอยู่ฝั่งเซิร์ฟเวอร์และผูกกับตำแหน่งผู้เล่นจริง

| Event | ทิศทาง | หน้าที่ |
|---|---|---|
| `Hyper_Crafting:craft` | client → server | ขอคราฟ |
| `Hyper_Crafting:upgrade` | client → server | ขออัปเกรด |
| `Hyper_Crafting:cancel` | client → server | ยกเลิกงาน |
| `Hyper_Crafting:setVaultPref` | client → server | เลือกตู้ |
| `Hyper_Crafting:reqVaults` | client → server | ขอรายการตู้ |
| `Hyper_Crafting:reqVaultCounts` | client → server | ขอจำนวนของในตู้ |
| `Hyper_Crafting:progress` | server → client | เริ่มจับเวลา |
| `Hyper_Crafting:done` | server → client | จบงาน พร้อมผล |
| `Hyper_Crafting:cancelled` | server → client | ยืนยันการยกเลิก |
| `Hyper_Crafting:setVaults` | server → client | ส่งรายการตู้ |
| `Hyper_Crafting:vaultCounts` | server → client | ส่งจำนวนของในตู้ |
| `Hyper_Crafting:announce` | server → client | ประกาศทั้งเซิร์ฟตอนคราฟของหายากติด |

## อยากต่อ export เพิ่ม

ถ้าอยากให้ resource อื่นสั่งงานโต๊ะนี้ได้ (เช่นเปิดโต๊ะจากเมนูอื่น) ต้องเพิ่ม export เองที่ `core/server/main.lua` หรือ `core/client/main.lua` ตอนนี้ยังไม่มีให้

ถ้าเพิ่ม ให้ตรวจสิทธิ์และระยะทางซ้ำในฟังก์ชัน export ด้วย อย่าพึ่งว่าผู้เรียกตรวจมาแล้ว
