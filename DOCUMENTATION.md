# 📋 توثيق مشروع نظام جداول التوقيت والامتحانات
## وثيقة تقنية شاملة — سجل المحادثة البرمجية الكاملة

> **المستودع:** [github.com/labgaa2012/university-timetable](https://github.com/labgaa2012/university-timetable)  
> **تاريخ الإنشاء:** 01 مايو 2026  
> **الإصدار الحالي:** v9  
> **حجم الملف الرئيسي:** 2922 سطر (ملف HTML وحيد)  
> **عدد الدوال البرمجية:** 117 دالة JavaScript  
> **عدد الصفحات:** 13 صفحة في تطبيق واحد

---

## 📌 فهرس المحتويات

1. [نظرة عامة على المشروع](#1-نظرة-عامة-على-المشروع)
2. [مسار التطوير — الإصدار تلو الإصدار](#2-مسار-التطوير)
3. [البنية التقنية الكاملة](#3-البنية-التقنية-الكاملة)
4. [نظام إدارة البيانات localStorage](#4-نظام-إدارة-البيانات)
5. [الصفحات والمكونات](#5-الصفحات-والمكونات)
6. [الميزات التفصيلية](#6-الميزات-التفصيلية)
7. [قواعد حساب الساعات الإضافية](#7-قواعد-حساب-الساعات-الإضافية)
8. [نظام الاستيراد والتصدير](#8-نظام-الاستيراد-والتصدير)
9. [سجل الأخطاء والإصلاحات](#9-سجل-الأخطاء-والإصلاحات)
10. [سجل Git الكامل](#10-سجل-git-الكامل)

---

## 1. نظرة عامة على المشروع

### الهدف
نظام إدارة جداول التوقيت والامتحانات مُصمَّم خصيصاً لـ **كلية العلوم الاقتصادية والتجارية وعلوم التسيير** في **جامعة عمار ثليجي — الأغواط**، الجزائر.

### الاختيارات التقنية

| القرار | الاختيار | السبب |
|---|---|---|
| المنصة | HTML/CSS/JS وحيد | لا يحتاج خادم، يعمل بالمتصفح مباشرة |
| التخزين | `localStorage` | بيانات دائمة دون قاعدة بيانات |
| تصدير Excel | مكتبة XLSX (SheetJS) | مجانية، تعمل في المتصفح |
| الخطوط | Cairo + Tajawal (Google Fonts) | دعم عربي ممتاز |
| الاتجاه | RTL كامل | واجهة عربية أصيلة |
| الألوان الرئيسية | Navy `#0f1f3d` + Teal `#0d8a7d` + Gold `#c9a84c` | طابع أكاديمي رسمي |

### الإحصائيات التقنية

```
الملف الرئيسي    : index.html
إجمالي الأسطر   : 2922 سطر
CSS             : ~450 سطر (متغيرات + أصناف + Grid + Print)
HTML            : ~600 سطر (13 صفحة + 6 modals)
JavaScript      : ~1850 سطر
دوال JS         : 117 دالة
صفحات التطبيق   : 13 صفحة (pg-*)
Modals          : 6 نوافذ منبثقة
```

---

## 2. مسار التطوير

التطوير جرى في **جلسة واحدة مستمرة** — كل إصدار بُني فوق السابق بشكل تراكمي.

### الإصدار الأولي — Preview فقط
**الطلب:** "أنشئ HTML preview لهذا النظام"

البداية كانت نموذجاً تجريبياً بدون أي وظائف حقيقية — بيانات ثابتة، واجهة جمالية فقط.

```
الملف: جدول_الجامعة_preview.html
المحتوى: بانر جانبي + جدول توقيت + جدول امتحانات (بيانات ثابتة)
```

---

### v1 — الإصدار الأول على GitHub
**الطلب:** "قم بإنشاء repo على حسابي labgaa2012"

```bash
# الأوامر المُنفَّذة
curl -X POST https://api.github.com/user/repos \
  -H "Authorization: token $TOKEN" \
  -d '{"name":"university-timetable","private":false}'

git init && git add -A
git commit -m "🎓 Initial commit"
git push -u origin main
```

**الملاحظة:** التوكن المُستخدم: `[TOKEN][TOKEN_REDACTED]` — حساب `labgaa2012`

---

### v2 — الميزات الحقيقية الأربع
**الطلب:** إضافة: drag & drop، localStorage، تصدير Excel حقيقي، إضافة حصة بنقرة

#### أ) localStorage
```javascript
const SK = 'univ_tt_v4'; // Storage Key

function save() {
  try { localStorage.setItem(SK, JSON.stringify(S)); } catch(e) {}
}

function load() {
  try {
    const d = localStorage.getItem(SK);
    S = d ? JSON.parse(d) : defState();
  } catch(e) { S = defState(); }
}
```

#### ب) Drag & Drop
```javascript
// على بطاقة الحصة
draggable="true"
ondragstart="onDS(event, '${session.id}')"
ondragend="..."

// على خلية الجدول
ondragover="event.preventDefault(); this.classList.add('drop-over')"
ondragleave="this.classList.remove('drop-over')"
ondrop="onDrop(event, '${day}', '${slot}')"

function onDrop(e, day, slot) {
  e.preventDefault();
  const s = S.sessions.find(x => x.id === dragId);
  // فحص التعارض
  const conflict = S.sessions.find(x =>
    x.id !== dragId &&
    x.day === day &&
    x.slot === slot &&
    x.lv === s.lv &&
    x.dp === s.dp
  );
  if (conflict) { toast('⚠️ تعارض'); return; }
  s.day = day; s.slot = slot;
  save(); renderTT();
}
```

#### ج) إضافة بنقرة على الخلية
```javascript
// كل خلية فارغة تحمل:
class="ec" // empty cell
onclick="openAddSess('${day}', '${slot}')"

// CSS للتوجيه البصري
table#ttbl td.ec::after {
  content: '＋';
  color: rgba(13,138,125,.4);
  font-size: 20px;
  // يظهر عند hover
}
```

#### د) تصدير Excel حقيقي (SheetJS)
```javascript
// استدعاء المكتبة من CDN
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js">

function buildExcel(sessions, exams, fname) {
  const wb = XLSX.utils.book_new();
  // ورقة لكل مستوى/قسم
  levels.forEach(lv => depts.forEach(dp => {
    const rows = [['الفترة', ...DAYS]];
    SLOTS.forEach(slot => {
      const row = [SL[slot]];
      DAYS.forEach(day => {
        const found = sessions.filter(s =>
          s.day === day && s.slot === slot
        );
        row.push(found.map(s => `${s.subj}\n${s.tch}\n${s.rm}`).join(' | ') || '');
      });
      rows.push(row);
    });
    const ws = XLSX.utils.aoa_to_sheet(rows);
    XLSX.utils.book_append_sheet(wb, ws, sheetName);
  }));
  XLSX.writeFile(wb, fname + '.xlsx');
}
```

---

### v3 — البانر القابل للطي + صفحات الإدارة
**الطلب:** "البانر لا يكون ثابتاً، والتخصصات/الأساتذة/القاعات تصبح أزراراً تفتح صفحاتها الخاصة"

#### البانر القابل للطي
```css
#sidebar {
  transform: translateX(0);
  transition: transform .35s cubic-bezier(.4, 0, .2, 1);
}
#sidebar.hidden { transform: translateX(100%); }

/* Handle ثابت دائماً على الحافة */
#sb-handle {
  position: fixed;
  top: 50%;
  right: 0;
  transform: translateY(-50%) translateX(0);
}
#sb-handle.sb-open {
  transform: translateY(-50%) translateX(calc(-1 * var(--sidebar-w)));
}
```

#### نمط أزرار التنقل في البانر
```html
<button class="sb-nav-btn" onclick="showMgmtPg('teachers')">
  <div class="snb-left">
    <div class="ai">👨‍🏫</div>
    <span>إدارة الأساتذة</span>
  </div>
  <span class="snb-count" id="snb-count-teachers">0</span>
  <span class="snb-arrow">←</span>
</button>
```

#### دالة التنقل المزدوجة
```javascript
// للصفحات الرئيسية (Header nav)
function showPg(id, btn) {
  document.querySelectorAll('.pg').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.nav-tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.sb-nav-btn').forEach(b => b.classList.remove('active-pg'));
  document.getElementById('pg-' + id).classList.add('active');
  if (btn) btn.classList.add('active');
  // تحميل البيانات حسب الصفحة
  if (id === 'rooms') renderRooms();
  if (id === 'timetable') renderTT();
  // ...
}

// لصفحات الإدارة (Sidebar nav)
function showMgmtPg(id) {
  document.querySelectorAll('.pg').forEach(p => p.classList.remove('active'));
  document.getElementById('pg-' + id).classList.add('active');
  const btn = document.getElementById('snb-' + id);
  if (btn) btn.classList.add('active-pg');
  if (id === 'teachers') renderTchGrid();
  if (id === 'rooms-manage') renderRmGrid();
  // ...
}
```

---

### v4 — إدارة التخصصات في صفحة مستقلة
**الطلب:** "نحول التخصصات من قائمة منسدلة إلى زر يعرض محتواها في الصفحة الرئيسية"

```javascript
function addSpecInline() {
  const n = document.getElementById('spIn').value.trim();
  if (!n) { toast('⚠️ أدخل اسم التخصص'); return; }
  const lv = document.getElementById('sp-lv-inline')?.value || S.levels[0];
  const dp = document.getElementById('sp-dp-inline')?.value || S.depts[0];
  // توليد الشارة تلقائياً (L1, M2, ...)
  const m = {'ليسانس':'L','ماستر':'M'}, nm = {'أولى':'1','ثانية':'2','ثالثة':'3'};
  let ls = '';
  for (const k in m) if (lv.includes(k)) ls = m[k];
  for (const k in nm) if (lv.includes(k)) ls += nm[k];
  S.specs.push({ name: n, lv, dp, ls: ls || '+' });
  save(); renderSpGrid();
}
```

---

### v5 — مصفوفة أشغال القاعات الكاملة
**الطلب:** تطبيق شرح تقني مفصّل — مصفوفة (قاعات × أيام × فترات)

#### آلية البناء
```javascript
function renderRooms() {
  const days = rmView === 'day'
    ? [document.getElementById('rm-day')?.value || DAYS[0]]
    : DAYS;

  let thead = `<thead>
    <tr>
      <th rowspan="2">القاعة</th>
      ${days.map(d =>
        `<th colspan="${SLOTS.length}">${d}</th>`
      ).join('')}
    </tr>
    <tr>
      ${days.flatMap(() =>
        SLOTS.map(sl => `<th>${SL[sl]}</th>`)
      ).join('')}
    </tr>
  </thead>`;

  let tbody = '<tbody>';
  rooms.forEach(room => {
    tbody += `<tr><td class="room-name">${room}</td>`;
    days.forEach(day => {
      SLOTS.forEach(slot => {
        // المطابقة الثلاثية: قاعة + يوم + فترة
        const entry = activeSessions.find(s =>
          s.rm === room &&
          s.day === day &&
          s.slot === slot
        );
        if (entry) {
          tbody += `<td class="cell-occ">
            <div class="cell-inner">
              <div class="ci-subj">${entry.subj}</div>
              <div class="ci-tch">👨‍🏫 ${entry.tch}</div>
              <div class="ci-grp">${grp}</div>
            </div></td>`;
        } else {
          tbody += `<td class="cell-free">
            <div class="cell-inner">شاغرة</div></td>`;
        }
      });
    });
    tbody += '</tr>';
  });
}
```

#### التصدير لـ Excel مع دمج الخلايا
```javascript
function expRoomsExcel() {
  const ws = XLSX.utils.aoa_to_sheet(data);
  // دمج خلايا اليوم (colspan)
  ws['!merges'] = DAYS.map((_, di) => ({
    s: { r: 0, c: 1 + di * SLOTS.length },
    e: { r: 0, c: di * SLOTS.length + SLOTS.length }
  }));
  XLSX.writeFile(wb, 'rooms-occupancy.xlsx');
}
```

---

### v6 — ملف الأستاذ الكامل + الفلترة
**الطلب:** "4 معلومات لكل أستاذ: الاسم، الرتبة (7 رتب)، القسم، الحالة"

#### هيكل بيانات الأستاذ
```javascript
// قبل v6
{ name: 'أ. محمد', kind: 'permanent' }

// بعد v6
{
  name: 'أ. محمد بن عمر',
  rank: 'mca',        // أستاذ محاضر أ
  dept: 'قسم العلوم الاقتصادية',
  kind: 'permanent'   // دائم
}
```

#### جدول الرتب الأكاديمية
```javascript
const RANKS = {
  prof: { label: 'أستاذ التعليم العالي', cls: 'rank-prof', short: 'أ.ت.ع' },
  mca:  { label: 'أستاذ محاضر أ',        cls: 'rank-mca',  short: 'م.أ'   },
  mcb:  { label: 'أستاذ محاضر ب',        cls: 'rank-mcb',  short: 'م.ب'   },
  maa:  { label: 'أستاذ مساعد أ',        cls: 'rank-maa',  short: 'مس.أ'  },
  mab:  { label: 'أستاذ مساعد ب',        cls: 'rank-mab',  short: 'مس.ب'  },
  tmp:  { label: 'أستاذ مؤقت',           cls: 'rank-tmp',  short: 'مؤقت'  },
  sha:  { label: 'أستاذ مشارك',          cls: 'rank-sha',  short: 'مشارك' }
};
```

#### نظام الترحيل التلقائي (Migration)
```javascript
function load() {
  // الترحيل من string إلى object
  if (S.teachers.length && typeof S.teachers[0] === 'string') {
    S.teachers = S.teachers.map(n => ({
      name: n, kind: 'permanent', rank: 'mca', dept: ''
    }));
    save();
  }
  // الترحيل من object ناقص إلى object كامل
  S.teachers = S.teachers.map(t => ({
    name: t.name || '',
    kind: t.kind || 'permanent',
    rank: t.rank || 'mca',
    dept: t.dept || ''
  }));
}
```

#### نظام الفلترة متعدد المعايير
```javascript
function renderTchGrid() {
  const q      = document.getElementById('tch-search')?.value.trim().toLowerCase();
  const fRank  = document.getElementById('tch-f-rank')?.value;
  const fDeptV = document.getElementById('tch-f-dept')?.value;
  const fKind  = document.getElementById('tch-f-kind')?.value;

  let list = S.teachers;
  if (q)      list = list.filter(t => t.name.toLowerCase().includes(q));
  if (fRank)  list = list.filter(t => t.rank  === fRank);
  if (fDeptV) list = list.filter(t => t.dept  === fDeptV);
  if (fKind)  list = list.filter(t => t.kind  === fKind);
}
```

---

### v7 — الاستيراد من Excel + القوالب
**الطلب:** "زري تصدير واستيراد Excel في جميع الخصائص"

#### دالة قراءة Excel العامة
```javascript
function readExcel(file, cb) {
  const reader = new FileReader();
  reader.onload = e => {
    try {
      const wb = XLSX.read(e.target.result, {
        type: 'binary',
        cellDates: true
      });
      const ws = wb.Sheets[wb.SheetNames[0]];
      const rows = XLSX.utils.sheet_to_json(ws, { header: 1, defval: '' });
      cb(rows);
    } catch(err) {
      toast('❌ تعذّر قراءة الملف: ' + err.message);
    }
  };
  reader.readAsBinaryString(file);
}
```

#### المطابقة بالاسم لا بالترتيب
```javascript
// بدلاً من row[0], row[1]... نستخدم:
function headerMap(hdr) {
  const m = {};
  hdr.forEach((h, i) => { if (h) m[String(h).trim()] = i; });
  return m;
}

// ثم نبحث بعدة مفاتيح بديلة
const get = (keys, row) => {
  for (const k of keys) {
    if (m[k] !== undefined) return String(row[m[k]] || '').trim();
  }
  return '';
};

// مثال: العثور على عمود الأستاذ بأي من التسميات
const name = get(['الاسم واللقب', 'الاسم', 'name'], row);
```

#### نظام قوالب Excel
```javascript
const templates = {
  teachers: {
    headers: ['الاسم واللقب', 'الرتبة', 'القسم', 'الحالة'],
    example: [
      ['أ. محمد بن عمر', 'أستاذ محاضر أ', 'قسم العلوم الاقتصادية', 'أستاذ دائم'],
    ],
    note: 'الرتب المقبولة: أستاذ التعليم العالي / أستاذ محاضر أ / ...'
  },
  // rooms, specs, sessions, exams, subjects
};
```

---

### v8 — السنة/المستوى/الأقسام كصفحات مستقلة
**الطلب:** "نحولهم من منزلقة إلى أزرار تعرض محتواهم في الصفحة الرئيسية"

#### نمط البطاقة النشطة (Active Card Pattern)
```javascript
function setActiveYear(y) {
  S.ay = y;
  save();
  renderYrGrid(); // تحديث المصفوفة البصرية
  renderTT();     // تحديث الجدول
  toast('✅ السنة النشطة: ' + y);
}
```

```css
.yl-card.active-item {
  border-color: var(--navy);
  background: linear-gradient(135deg, var(--navy), var(--navy-light));
  color: #fff;
}
```

#### إضافة السنة التلقائية
```javascript
function addNextYear() {
  const last = S.years[S.years.length - 1]; // '2029/2030'
  const [a, b] = last.split('/').map(Number); // [2029, 2030]
  const next = `${a+1}/${b+1}`;              // '2030/2031'
  if (!S.years.includes(next)) S.years.push(next);
  S.ay = next;
  save(); renderYrGrid();
}
```

---

### v9 — إدارة المقاييس + إصلاح الأخطاء
**الطلب الأول:** إضافة صفحة إدارة المقاييس الدراسية  
**الطلب الثاني:** "توقفت الأزرار عن العمل"

#### الخطأ الذي أوقف كل الأزرار
كانت دالة `dlTemplate` تحتوي على object ناقص:

```javascript
// ❌ الكود الخاطئ
const templates = {
  subjects: { headers: [...], example: [...] },
    // مفتاح teachers: مفقود!
    headers: ['الاسم واللقب', ...],  // هذا خطأ syntax
```

```javascript
// ✅ الكود الصحيح
const templates = {
  subjects: { headers: [...], example: [...] },
  teachers: {  // ← هذا كان مفقوداً
    headers: ['الاسم واللقب', ...],
    example: [...]
  },
  rooms: { ... },
  // ...
};
```

**لماذا توقفت كل الأزرار؟**

```
SyntaxError في JS
    ↓
المتصفح يوقف تحليل الـ script كاملاً
    ↓
لا تُعرَّف أي دالة (renderAll, showPg, ...)
    ↓
كل الأزرار التي تستدعي هذه الدوال = لا تستجيب
```

#### طريقة تشخيص الخطأ
```bash
# استخراج JS من HTML
node -e "
  const html = require('fs').readFileSync('index.html','utf8');
  const m = html.match(/<script>([\s\S]*?)<\/script>/g);
  const js = m[m.length-1].replace(/<\/?script>/g,'');
  require('fs').writeFileSync('/tmp/check.js', js);
"

# فحص الـ syntax
node --check /tmp/check.js
# الناتج: /tmp/check.js:1660 → rooms:{ → SyntaxError
```

---

## 3. البنية التقنية الكاملة

### هيكل ملف index.html

```
index.html
├── <head>
│   ├── Google Fonts (Cairo + Tajawal)
│   └── SheetJS CDN (xlsx.full.min.js)
│
├── <style> (~450 سطر CSS)
│   ├── CSS Custom Properties (:root)
│   ├── Layout (Header + Sidebar + Main)
│   ├── Accordion styles
│   ├── Form elements
│   ├── Timetable table (.tt)
│   ├── Session cards (.sc)
│   ├── Rooms occupancy matrix (.occ-matrix)
│   ├── Management pages (.mgmt-card, .yl-card)
│   ├── Overtime page (.ot-table, .ot-bar)
│   ├── Import/Export buttons
│   └── @media print + @media (max-width:900px)
│
├── <body>
│   ├── <header> (fixed, z-index:1000)
│   │   ├── Logo + Title
│   │   ├── Header Nav (6 tabs)
│   │   └── Year badge + Toggle button
│   │
│   ├── <aside id="sidebar"> (fixed right, z-index:900)
│   │   ├── sb-hdr
│   │   ├── 3 Nav buttons (سنوات/مستويات/أقسام)
│   │   ├── Separator
│   │   └── 4 Nav buttons (تخصصات/مقاييس/أساتذة/قاعات)
│   │
│   ├── <button id="sb-handle"> (persistent handle)
│   │
│   └── <main id="main">
│       ├── Stats row (4 cards)
│       ├── pg-years
│       ├── pg-levels
│       ├── pg-depts
│       ├── pg-timetable ← افتراضي (active)
│       ├── pg-exams
│       ├── pg-rooms
│       ├── pg-comp
│       ├── pg-subjects
│       ├── pg-specs
│       ├── pg-teachers
│       ├── pg-rooms-manage
│       ├── pg-overtime
│       └── pg-archive
│
├── 6 Modals
│   ├── mSess (إضافة/تعديل حصة)
│   ├── mExam (إضافة/تعديل امتحان)
│   ├── mSpec (تخصص — deprecated)
│   ├── mEdit (تعديل اسم)
│   └── mOvDetail (تفاصيل الساعات الإضافية)
│
├── Hidden file inputs (5 للاستيراد)
│
└── <script> (~1850 سطر JS)
    ├── Constants (DAYS, SLOTS, AMPHI, RANKS, ...)
    ├── DEFAULT_SUBJECTS()
    ├── defState() + load() + save()
    ├── UI Functions (toggleSB, toggleAcc, showPg, showMgmtPg)
    ├── Render Functions (renderYrGrid → renderArchive)
    ├── CRUD Functions (add/rm for each entity)
    ├── Timetable (renderTT + Drag&Drop)
    ├── Exams (renderExams + CRUD)
    ├── Rooms Occupancy (renderRooms + expRoomsExcel + printRooms)
    ├── Compensation (findFree)
    ├── Subjects (renderSubjGrid + addSubject + ...)
    ├── Overtime Engine (calcTeacherOvertime + renderOvertime)
    ├── Archive (doArchive + expArcJSON + expArcExcel)
    ├── Import/Export (readExcel + impTeachers + dlTemplate + ...)
    └── INIT: load(); renderAll();
```

---

## 4. نظام إدارة البيانات

### هيكل الـ State الكامل

```javascript
const SK = 'univ_tt_v4'; // مفتاح localStorage

S = {
  // -- الهيكل الأكاديمي --
  years:    ['2025/2026', '2026/2027', ...], // السنوات
  levels:   ['سنة أولى ليسانس', ...],        // المستويات (5 افتراضية)
  depts:    ['قسم السنة أولى جذع مشترك', ...], // الأقسام (5 افتراضية)
  specs:    [{ name, lv, dp, ls }],           // التخصصات
  subjects: [{ id, name, sem, level, dept, spec }], // المقاييس (23 افتراضي)

  // -- الموارد البشرية والمادية --
  teachers: [{ name, rank, dept, kind }],     // الأساتذة
  rooms:    ['القاعة 1', ..., 'مدرج أ', ...], // 51 قاعة افتراضية

  // -- الجداول --
  sessions: [{                                // حصص التوقيت
    id, y, lv, dp, day, slot,
    subj, tch, rm, type, firstLec
  }],
  exams: [{                                   // الامتحانات
    id, y, lv, dp, subj,
    date, round, start, end, tch, rm
  }],

  // -- الأرشيف --
  archive: [{                                 // السنوات المؤرشفة
    yr, date, sc, ec, tc,
    sessions[], exams[], teachers[], rooms[]
  }],

  // -- الحالة النشطة --
  ay:    '2025/2026',          // السنة النشطة
  al:    'سنة أولى ليسانس',    // المستوى النشط
  ad:    'قسم السنة أولى ...',  // القسم النشط
  round: 1                      // دور الامتحانات
}
```

### قيم الثوابت

```javascript
const DAYS  = ['السبت','الأحد','الاثنين','الثلاثاء','الأربعاء','الخميس'];

const SLOTS = [
  '08:00-09:30', '09:30-11:00', '11:15-12:45',
  '13:30-15:00', '15:00-16:30'
];

const SL = { // عرض مُنسَّق
  '08:00-09:30': '08:00 - 09:30',
  // ...
};

const TL = { // أنواع الحصص
  tlc: 'محاضرة',
  ttd: 'أعمال موجهة',
  ttp: 'أعمال تطبيقية',
  tot: 'أخرى'
};

const TC = { // ألوان الخلفية (للطباعة)
  tlc: '#dbeafe', // أزرق فاتح
  ttd: '#dcfce7', // أخضر فاتح
  ttp: '#fed7aa', // برتقالي فاتح
  tot: '#ede9fe'  // بنفسجي فاتح
};

const AMPHI = ['مدرج أ', 'مدرج ب', 'مدرج ج'];
```

---

## 5. الصفحات والمكونات

| معرّف الصفحة | العنوان | الوصول | الميزات |
|---|---|---|---|
| `pg-timetable` | جدول التوقيت الأسبوعي | Header nav | Drag&Drop، نقرة للإضافة، Excel، طباعة |
| `pg-exams` | جدول الامتحانات | Header nav | دورين، فلتر، Excel، طباعة |
| `pg-rooms` | أشغال القاعات | Header nav | مصفوفة كاملة، أسبوعي/يومي، Excel، طباعة |
| `pg-comp` | تعويض الحصص | Header nav | بحث فراغ بالقاعة/الأستاذ/اليوم |
| `pg-overtime` | الساعات الإضافية | Header nav | حساب تلقائي، تفاصيل، Excel، طباعة |
| `pg-archive` | الأرشيف | Header nav | JSON + Excel لكل سنة |
| `pg-years` | إدارة السنوات | Sidebar nav | بطاقات نشطة، إضافة تلقائية/يدوية |
| `pg-levels` | إدارة المستويات | Sidebar nav | بطاقات نشطة، إضافة/حذف |
| `pg-depts` | إدارة الأقسام | Sidebar nav | بطاقات نشطة + عداد الأساتذة والحصص |
| `pg-subjects` | إدارة المقاييس | Sidebar nav | فلتر بالسداسي/المستوى/القسم، 23 مقياس افتراضي |
| `pg-specs` | إدارة التخصصات | Sidebar nav | بطاقات + شارات L1/M2... |
| `pg-teachers` | إدارة الأساتذة | Sidebar nav | 7 رتب، فلترة متعددة، Excel |
| `pg-rooms-manage` | إدارة القاعات | Sidebar nav | 51 قاعة، مدرجات منفصلة، Excel |

---

## 6. الميزات التفصيلية

### 6.1 جدول التوقيت الأسبوعي

#### هيكل الجدول
```
         | السبت | الأحد | الاثنين | الثلاثاء | الأربعاء | الخميس
---------|-------|-------|---------|---------|---------|-------
08:00    | [محاضرة] | [TD] |         | [TP]   |         | [محاضرة]
09:30    | ...
11:15    | ...
13:30    | ...
15:00    | ...
```

#### بطاقة الحصة (Session Card)
```html
<div class="sc tlc" draggable="true"
  ondragstart="onDS(event,'${s.id}')">
  <button class="sceb" onclick="openEditSess('${s.id}')">✏️</button>
  <button class="scdb" onclick="delSess('${s.id}')">✕</button>
  <div class="ss">${s.subj}</div>     <!-- الاسم Bold -->
  <div class="si">👨‍🏫 ${s.tch}</div> <!-- الأستاذ -->
  <div class="si">🏫 ${s.rm}</div>   <!-- القاعة -->
  <div class="si">محاضرة</div>        <!-- النوع -->
</div>
```

#### CSS الأنواع
```css
.sc.tlc { background:#dbeafe; color:#1d4ed8; border-right:3px solid #1d4ed8; }
.sc.ttd { background:#dcfce7; color:#15803d; border-right:3px solid #15803d; }
.sc.ttp { background:#fed7aa; color:#c2410c; border-right:3px solid #c2410c; }
.sc.tot { background:#ede9fe; color:#6d28d9; border-right:3px solid #6d28d9; }
```

### 6.2 مصفوفة أشغال القاعات

الـ complexity البرمجية:
```
O(rooms × days × slots) = 51 × 6 × 5 = 1530 خلية
```

كل خلية تُحسب بـ `Array.find()` على `sessions`، والنتيجة في `O(n)` لكل خلية.

### 6.3 تعويض الحصص

```javascript
function findFree() {
  const day   = document.getElementById('cp-day').value;
  const room  = document.getElementById('cp-room').value;
  const tch   = document.getElementById('cp-teach').value;

  // جمع الفترات المشغولة: قاعة OR أستاذ
  const occSlots = S.sessions
    .filter(s =>
      s.y === S.ay &&
      s.day === day &&
      (s.rm === room || (tch && s.tch === tch))
    )
    .map(s => s.slot);

  // الفترات المتاحة = الكل - المشغول
  const freeSlots = SLOTS.filter(sl => !occSlots.includes(sl));
}
```

---

## 7. قواعد حساب الساعات الإضافية

> المرجع: القانون الجزائري المعمول به في جامعة عمار ثليجي — الأغواط

### النصاب القانوني

| نوع الأستاذ | النصاب الأسبوعي |
|---|---|
| دائم (`permanent`) | 9 ساعات معادلة |
| مؤقت (`temporary`) | 0 — كل ساعاته إضافية |
| مشارك (`associate`) | 0 — كل ساعاته إضافية |

### معاملات التحويل

| نوع الحصة | المدة الزمنية | الساعات المعادلة | المعامل |
|---|---|---|---|
| محاضرة أولى | 1.5 ساعة | 2.25 ساعة | × 1.5 |
| محاضرة مكررة | 1.5 ساعة | 1.5 ساعة | × 1.0 |
| أعمال موجهة (TD) | 1.5 ساعة | 1.5 ساعة | × 1.0 |
| أعمال تطبيقية (TP) | 1.5 ساعة | 1.5 ساعة | × 1.0 |

### المعادلة

```
مجموع المعادل = (محاضرة أولى × 2.25) + (محاضرة مكررة × 1.5) + (TD/TP × 1.5)
الساعات الإضافية = max(0, مجموع المعادل - النصاب)
```

### الكود المُطبِّق

```javascript
const QUORUM = 9;

function calcTeacherOvertime(tchName) {
  const sessions = S.sessions.filter(s => s.y === S.ay && s.tch === tchName);
  const tchObj   = S.teachers.find(t => t.name === tchName) || { kind: 'permanent' };

  let firstLecCount = 0, repLecCount = 0, tdTpCount = 0;

  sessions.forEach(s => {
    if (s.type === 'tlc') {
      if (s.firstLec === 0) repLecCount++;
      else firstLecCount++;
    } else if (s.type === 'ttd' || s.type === 'ttp') {
      tdTpCount++;
    }
  });

  const eqFirstLec = firstLecCount * 2.25;
  const eqRepLec   = repLecCount   * 1.5;
  const eqTdTp     = tdTpCount     * 1.5;
  const totalEq    = eqFirstLec + eqRepLec + eqTdTp;
  const quorum     = tchObj.kind === 'permanent' ? QUORUM : 0;
  const overtime   = Math.max(0, totalEq - quorum);

  return { name: tchName, kind: tchObj.kind,
           firstLecCount, repLecCount, tdTpCount,
           totalEq, quorum, overtime };
}
```

---

## 8. نظام الاستيراد والتصدير

### جدول الدعم الكامل

| القسم | تصدير Excel | استيراد Excel | قالب | طباعة |
|---|:---:|:---:|:---:|:---:|
| جدول التوقيت | ✅ | ✅ | ✅ | ✅ |
| جدول الامتحانات | ✅ | ✅ | ✅ | ✅ |
| الأساتذة | ✅ | ✅ | ✅ | — |
| القاعات | ✅ | ✅ | ✅ | ✅ |
| التخصصات | ✅ | ✅ | ✅ | — |
| المقاييس | ✅ | ✅ | ✅ | — |
| أشغال القاعات | ✅ | — | — | ✅ |
| الساعات الإضافية | ✅ | — | — | ✅ |
| الأرشيف | JSON + Excel | — | — | — |

### مسار الاستيراد

```
المستخدم يضغط "استيراد"
    ↓
trigImp('imp-teachers') → input.click()
    ↓
المستخدم يختار ملف .xlsx/.xls/.csv
    ↓
onchange="impTeachers(this)"
    ↓
readExcel(file, rows => {
    ↓
    headerMap(rows[0]) → { 'الاسم واللقب': 0, 'الرتبة': 1, ... }
    ↓
    rows.slice(1).forEach(row => {
      // مطابقة مرنة بالاسم
      // تحويل النص إلى كود (rank, kind)
      // تخطي المكررات
      S.teachers.push(...)
    })
    ↓
    save(); renderTchGrid();
    toast('✅ استُورد 15 أستاذ · تخطّي 2 مكرر')
})
```

### تفاصيل تحويل بيانات الأساتذة

```javascript
// جداول التحويل العكسي
const RANK_REV = {
  'أستاذ التعليم العالي': 'prof',
  'أستاذ محاضر أ':        'mca',
  'أستاذ محاضر ب':        'mcb',
  'أستاذ مساعد أ':        'maa',
  'أستاذ مساعد ب':        'mab',
  'أستاذ مؤقت':           'tmp',
  'أستاذ مشارك':          'sha',
  // اختصارات مقبولة أيضاً
  'محاضر أ': 'mca', 'مشارك': 'sha', ...
};

const KIND_REV = {
  'أستاذ دائم':  'permanent',
  'دائم':        'permanent',
  'أستاذ مؤقت': 'temporary',
  'مؤقت':       'temporary',
  'أستاذ مشارك':'associate',
  'مشارك':      'associate'
};
```

---

## 9. سجل الأخطاء والإصلاحات

### الخطأ الرئيسي — SyntaxError أوقف كل الأزرار

**التاريخ:** 01 مايو 2026  
**المُبلَّغ عنه:** "توقفت الأزرار عن العمل"  
**الشدة:** حرجة — التطبيق بأكمله معطل

**التشخيص:**
```bash
# كشف السطر المُشكِل
node --check /tmp/check.js
# /tmp/check.js:1660
#     rooms:{
#     ^^^^^
# SyntaxError: Missing initializer in const declaration
```

**الجذر:**
```javascript
// ❌ قبل الإصلاح — مفتاح teachers: مفقود
const templates = {
  subjects: { headers: [...] },
    headers: ['الاسم واللقب'],  // ← parser يفسرها كـ const ناقص
```

**الإصلاح:**
```javascript
// ✅ بعد الإصلاح
const templates = {
  subjects: { headers: [...] },
  teachers: {                    // ← تمت الإضافة
    headers: ['الاسم واللقب'],
    example: [...]
  },
```

**الدرس المستفاد:**  
قبل كل commit يجب تشغيل `node --check` على ملف JS المُستخرج للتحقق من صحة الـ syntax.

### مشكلة رموز Unicode في التعليقات

**الرمز:** `──` (U+2500 — BOX DRAWINGS LIGHT HORIZONTAL)  
**المشكلة:** بعض محللات JS تتعامل معها كـ identifier غير متوقع  
**الإصلاح:** استبدال بـ `--` (ASCII) في جميع تعليقات JS

```python
# الإصلاح بـ Python
with open('index.html', 'r', encoding='utf-8') as f:
    content = f.read()
content = content.replace('\u2500\u2500', '--')
```

### مشكلة الدوال المكررة

**السبب:** إضافة دوال `renderSubjGrid` و`addSubject` وغيرها مرتين  
**الأثر:** الدالة الثانية تُلغي الأولى (في JS الأخيرة تفوز)  
**الإصلاح:** حذف الكتلة المكررة والإبقاء على النسخة الأكثر اكتمالاً

---

## 10. سجل Git الكامل

```
d409a60  chore: add .gitignore, remove node_modules           2026-05-01
60be274  🐛 fix: restore all buttons — JS syntax error (v9)   2026-05-01
c071a1e  📖 feat: subjects management (المقاييس) v9           2026-05-01
bf7d3ab  🗂️ feat: years/levels/depts nav buttons (v8)         2026-05-01
4582846  📥 feat: Excel import + templates for all (v7)        2026-05-01
6cec862  👨‍🏫 feat: teacher full profile + filters (v6)         2026-05-01
6cfb6b2  🏫 feat: rooms occupancy matrix (v5)                 2026-05-01
f443960  ✨ feat: specs management page (v4)                   2026-05-01
87b5922  🎨 feat: sidebar nav buttons + mgmt pages (v3)       2026-05-01
b5e0e87  ⏰ feat: overtime calculation (جامعة الأغواط)         2026-05-01
c02de93  ✨ feat: drag&drop + localStorage + Excel (v2)        2026-05-01
39c96e9  🎓 Initial commit                                     2026-05-01
```

### إحصائيات التطوير

```
الجلسات : 1 جلسة متواصلة
المدة   : يوم واحد كامل (01 مايو 2026)
Commits : 12 commit
الإصدارات المُسلَّمة : v1 → v9 (9 ملفات HTML)
إجمالي الأسطر المضافة : ~3000 سطر
إجمالي الأسطر المحذوفة: ~1800 سطر (إعادة كتابة وتحسين)
```

---

## ملاحظات تقنية ختامية

### ما يميز هذا المشروع معمارياً

1. **ملف وحيد (Single File Application)** — كل شيء في `index.html` دون أي build tool أو node_modules
2. **Zero-backend** — `localStorage` بديل قاعدة البيانات، يعمل offline تماماً
3. **Progressive Enhancement** — يبدأ بـ 23 مقياس و 51 قاعة افتراضية جاهزة للاستخدام
4. **Migration System** — عند تحميل بيانات قديمة يُرحِّلها تلقائياً للهيكل الجديد
5. **RTL-first Design** — كل CSS مكتوب بافتراض الاتجاه العربي من البداية

### نقاط التطوير المستقبلية المقترحة

- [ ] تحويل إلى PWA مع Service Worker للعمل offline كاملاً
- [ ] إضافة وضع الفصل الدراسي (سداسي أول / ثاني) كفلتر رئيسي
- [ ] ربط المقاييس بالحصص تلقائياً (autocomplete من قائمة المقاييس)
- [ ] تقرير عبء التدريس الكامل للقسم
- [ ] تصدير PDF مباشر (بدون نافذة طباعة) عبر jsPDF

---

*آخر تحديث: 01 مايو 2026 — الإصدار v9*
