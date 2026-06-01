```markdown
# IG Unfollowers Checker

สคริปต์สำหรับหาบัญชีบน Instagram ที่ "เราติดตามเขา แต่เขาไม่ได้ติดตามเรากลับ" 

## 🛡 ความปลอดภัย & ข้อดี
- **ปลอดภัย 100%:** ไม่มีการขอรหัสผ่าน ไม่ต้องล็อกอินผ่านแอป Third-party 
- **Privacy:** โค้ดทำงานแบบ Local บนเบราว์เซอร์ของคุณเท่านั้น ไม่มีการส่งข้อมูลออกไปยังเซิร์ฟเวอร์อื่น
- **อัปเดตง่าย:** ดึงข้อมูลจากโครงสร้างหน้าเว็บ (DOM) โดยตรง ไม่พึ่งพา API ที่มักจะถูกจำกัดสิทธิ์โดย Instagram

---

## 🛠 วิธีใช้งาน

ใช้งานผ่านเบราว์เซอร์บนคอมพิวเตอร์ (Chrome / Edge / Safari)

### 1. โหลด Script
ไปที่หน้าโปรไฟล์ Instagram ของคุณ กด `F12` (หรือคลิกขวา > Inspect) เพื่อเปิด Developer Tools แล้วไปที่แท็บ **Console** จากนั้นก๊อปปี้โค้ดนี้ไปวางแล้วกด `Enter` 

```javascript
async function autoScrollAndCollect() {
    const dialog = document.querySelector('div[role="dialog"]');
    if (!dialog) {
        console.error("❌ ไม่พบหน้าต่างรายชื่อ กรุณาคลิกเปิดรายชื่อก่อนรันคำสั่ง");
        return [];
    }

    const extractUsernames = () => {
        const links = dialog.querySelectorAll('a[role="link"][href^="/"]');
        const usernames = new Set();
        links.forEach(link => {
            const username = link.getAttribute('href').replace(/\//g, '');
            if (username && username !== 'explorepeople' && !username.includes('?')) {
                usernames.add(username);
            }
        });
        return usernames;
    };

    let scrollContainer = null;
    const firstLink = dialog.querySelector('a[role="link"][href^="/"]');
    if (firstLink) {
        let currentEl = firstLink.parentElement;
        while (currentEl && currentEl !== dialog) {
            if (currentEl.scrollHeight > currentEl.clientHeight) {
                const style = window.getComputedStyle(currentEl);
                if (['auto', 'scroll', 'hidden'].includes(style.overflowY)) {
                    scrollContainer = currentEl;
                    break;
                }
            }
            currentEl = currentEl.parentElement;
        }
    }

    if (!scrollContainer) return console.error("❌ หาพื้นที่เลื่อนไม่เจอ ลองขยับหน้าต่างนิดนึงครับ"), [];

    console.log("⏳ กำลังโหลดข้อมูล... (ห้ามปิดหน้าต่างจนกว่าจะเสร็จ)");
    
    let items = extractUsernames();
    let isScrolling = true, noNewDataCount = 0; 

    while (isScrolling) {
        const beforeCount = items.size;
        scrollContainer.scrollTop = scrollContainer.scrollHeight;
        await new Promise(r => setTimeout(r, 1500)); 
        items = extractUsernames();
        
        if (items.size === beforeCount) {
            if (++noNewDataCount >= 3) isScrolling = false; 
        } else {
            noNewDataCount = 0; 
        }
    }
    
    const result = Array.from(items);
    console.log(`✅ พบรายชื่อทั้งหมด: ${result.length} บัญชี`);
    return result;
}

```

### 2. ดึงข้อมูล "กำลังติดตาม" (Following)

คลิกเปิดหน้าต่าง **กำลังติดตาม (Following)** บนหน้า IG ของคุณ จากนั้นวางคำสั่งนี้ใน Console แล้วกด `Enter` (รอจนกว่าระบบจะเก็บข้อมูลเสร็จ)

```javascript
window.myFollowing = await autoScrollAndCollect();

```

### 3. ดึงข้อมูล "ผู้ติดตาม" (Followers)

ปิดหน้าต่างเดิม แล้วคลิกเปิดหน้าต่าง **ผู้ติดตาม (Followers)** จากนั้นวางคำสั่งนี้ใน Console แล้วกด `Enter` (รอจนกว่าระบบจะเก็บข้อมูลเสร็จ)

```javascript
window.myFollowers = await autoScrollAndCollect();

```

### 4. เช็คคนไม่ฟอลกลับ & ส่งออกไฟล์ `.txt`

เมื่อได้ข้อมูลครบทั้ง 2 ส่วน ให้รันคำสั่งสุดท้ายนี้เพื่อเปรียบเทียบรายชื่อ พร้อมดาวน์โหลดไฟล์ `unfollowers.txt` เก็บไว้ดูในเครื่อง:

```javascript
const unfollowers = window.myFollowing.filter(user => !window.myFollowers.includes(user));
console.table(unfollowers);

// สั่งดาวน์โหลดเป็นไฟล์ .txt
const blob = new Blob([unfollowers.join('\n')], { type: 'text/plain' });
const link = document.createElement('a');
link.href = URL.createObjectURL(blob);
link.download = 'unfollowers.txt';
link.click();

```

---

## 📌 ข้อควรระวัง

เมื่อได้รายชื่อแล้ว แนะนำให้ค้นหาและกด Unfollow ด้วยตัวเองทีละคน **อย่ากดรัวๆ หรือใช้บอทกด Unfollow อัตโนมัติ** เพราะอาจทำให้บัญชีของคุณโดนบล็อกการกระทำ (Action Block) จากระบบป้องกันสแปมของ Instagram ได้

