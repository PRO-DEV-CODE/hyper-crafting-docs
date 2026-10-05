# การตั้งค่า

## ไฟล์ทั้งหมด

| ไฟล์ | ฝั่ง | ใช้ตั้งอะไร |
|---|---|---|
| `config/config.shared.lua` | shared | ค่ากลางทุกโต๊ะ · ระบบเลเวล · ตู้เซฟ · tooltip · ชื่อไอเทม |
| `config/config.craft.lua` | shared | หมวดหมู่และสูตรคราฟทั้งหมด |
| `config/config.upgrade.lua` | shared | หมวดและรายการอัปเกรด |
| `config/config.tables.lua` | shared | จุดโต๊ะทุกตัวในแผนที่ |
| `config/config.language.lua` | shared | ข้อความทุกบรรทัดใน UI และแจ้งเตือน (76 คีย์) |
| `config/config.client.lua` | client | ปุ่มเปิด · ขนาด UI · ธีม · marker · แอนิเมชัน |
| `config/config.server.lua` | server | เว็บฮุก Discord (อ่านจาก convar) |
| `config/framework/esx/*.lua` | ทั้งสาม | ตัวเชื่อม ESX — เปลี่ยนเฟรมเวิร์กแก้ที่นี่ที่เดียว |

## ค่าที่ปรับบ่อย

### ปุ่ม ขนาด และธีม — `config/config.client.lua`

```lua
Config.OpenControl = 38          -- ปุ่มเปิดโต๊ะ (38 = E)
Config.UiScale     = 0.92        -- ขนาด UI รวม (0.40–1.50)
Config.Theme       = 'seagreen'  -- ธีมสี (ดูหน้า "ธีมสี")
Config.ShowPrompt  = true        -- โชว์ Text UI ตอนเข้าใกล้
Config.UrlImage    = 'nui://Hyper_Inventory/html/img/items'
```

### ค่ากลาง — `config/config.shared.lua`

```lua
Shared.Settings = {
    maxAmount  = 10,    -- คราฟต่อครั้งสูงสุด (สูตรเขียนทับได้)
    busyLock   = true,  -- กันคราฟซ้อน — อย่าปิด
    cancelable = true,  -- ยกเลิกกลางคันได้ (คืนของ)
    useVault   = true,  -- ดึงวัตถุดิบจากตู้ Hyper_Vault
    vaultGroups  = { { group = 'STANDARD', label = 'ตู้เซฟส่วนตัว' } },
    defaultVault = 'STANDARD',
}

Shared.Leveling = {
    enabled    = true,
    xpPerLevel = 100,   -- xp ต่อ 1 เลเวล
    defaultXp  = 5,     -- xp ต่อชิ้นถ้าสูตรไม่กำหนด
    maxLevel   = 100,
}
```

`vaultGroups[].group` ต้องตรงกับ `Config.GroupVault` ของ `Hyper_Vault` ไม่งั้นตู้จะไม่ขึ้นในดรอปดาวน์

### หนึ่งสูตรคราฟ — `config/config.craft.lua`

```lua
Shared.Recipes[1] = {
    {
        item      = 'lockpick',       -- ไอเทมที่ได้ (หรือ WEAPON_* ถ้า isWeapon)
        label     = 'ล็อคพิค',
        amount    = 1,                -- ได้กี่ชิ้นต่อครั้ง
        time      = 10,               -- วินาทีต่อชิ้น
        rate      = 10,               -- โอกาสสำเร็จ %
        maxAmount = 5,                -- คราฟทีละกี่ชิ้นสูงสุด
        reqLevel  = 0,                -- เลเวลขั้นต่ำ
        xp        = 25,               -- xp ต่อชิ้นสำเร็จ
        equipment = { 'card_work' },  -- ต้องมี แต่ไม่ถูกหัก
        cost      = { money = 2000 }, -- money / bank / black_money
        blueprint = {                 -- วัตถุดิบที่หายไป
            { item = 'aed', count = 3 },
            { item = 'afk', count = 2 },
        },
        failTo    = {},               -- ได้คืนถ้าพลาด
        announce  = false,            -- ประกาศทั้งเซิร์ฟตอนสำเร็จ
        rateBonus = {                 -- false = สูตรนี้ใช้ไม่ได้
            { item = 'aed', label = 'น้ำยาเร่งคราฟ', add = 5, maxStack = 10 },
        },
        antiBonus = {
            { item = 'aed', label = 'ตัวกันคราฟแตก' },
        },
    },
}
```

### หลายวิธีคราฟ

```lua
methods = {
    { blueprint = { { item = 'aed', count = 3 } }, cost = { money = 5000 } },  -- I
    { blueprint = { { item = 'iron', count = 6 } }, cost = { bank  = 4000 } }, -- II
}
```

### หนึ่งรายการอัปเกรด — `config/config.upgrade.lua`

```lua
{
    cat        = 'poolcue',           -- คีย์ใน Shared.UpgradeCategories
    base       = 'lockpick',          -- ของฐาน (ถูกหัก 1 ชิ้น)
    result     = 'lockpick',          -- ของที่ได้เมื่อสำเร็จ
    label      = 'Pool Cue II',
    time       = 8,
    rate       = 60,                  -- โอกาสสำเร็จ %
    cost       = { money = 150000 },
    equipment  = { 'card_work' },     -- ต้องมี ไม่ถูกหัก
    materials  = { { item = 'afk', count = 2 } },   -- หักทั้งสำเร็จและพลาด
    keepBaseOnFail = false,           -- false = พลาดเสียของฐาน
    antiBonus  = { { item = 'aed', label = 'ตัวกันอัพเกรดแตก' } },
    reqLevel   = 0,
    xp         = 10,
}
```

### โต๊ะและจุดวาง — `config/config.tables.lua`

```lua
{
    label      = 'โต๊ะคราฟทั่วไป',
    type       = 'craft',             -- 'craft' หรือ 'upgrade'
    desc       = 'สำหรับผลิตของทั่วไป เท่านั้น',
    coords     = vector3(964.78, -3184.13, 13.40),
    radius     = 2.0,                 -- ระยะกดเปิด
    categories = { 1, 3 },            -- {} = เปิดทุกหมวด
    job        = nil,                 -- { 'police' } = จำกัดอาชีพ
    model      = `gr_prop_gr_bench_04b`,  -- nil = ไม่ spawn prop
    heading    = 120.0,
}
```

### Marker และแอนิเมชัน — `config/config.client.lua`

```lua
Config.Marker = {
    enabled = true,
    type    = 2,                              -- ลูกศร
    size    = vector3(0.25, 0.25, 0.2),
    color   = { r = 10, g = 163, b = 136, a = 180 },
    zOffset = 1.0,
    drawDistance = 12.0,
}

Config.Anim = {
    enabled = true,
    dict    = 'amb@world_human_hammering@male@base',
    name    = 'base',
}
```

### เว็บฮุก Discord — `server.cfg`

```
set hyper_crafting_webhook "https://discord.com/api/webhooks/..."
```

ว่างไว้ = ไม่ส่ง log ส่วน audit trail ยังส่งผ่าน `Hyper_Discordlogs` แยกต่างหาก

## ข้อความใน UI

ข้อความทุกบรรทัดอยู่ที่ `config/config.language.lua` รวม 76 คีย์ แบ่งเป็นหัวข้อหน้าจอ ป้ายในแผงข้อมูล ปุ่ม ป้ายบนการ์ด ยอดเงิน สถานะว่าง Text UI และข้อความแจ้งเตือน แก้ที่นี่ที่เดียวแล้วเปลี่ยนทั้งระบบ

คีย์ที่มี `%s` คือช่องเติมค่า เช่น `Lang.canCraftTimes = 'คราฟได้ %s ครั้ง'` ห้ามลบ `%s` ทิ้ง
