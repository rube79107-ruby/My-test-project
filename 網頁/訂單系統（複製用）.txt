/**
 * Feiyen 手繪｜訂單系統（前台需求表＋後台管理）
 *
 * 前台：客人填需求表，送出後自動存進試算表，並寄 Email 通知你
 * 後台：只有你能開，看每月統計、更新訂單狀態和報價金額
 *
 * 第一次使用：
 * 1. 貼上整份程式 → 儲存
 * 2. 函式選 setup → 執行 → 完成授權
 * 3. 部署 → 新增部署作業 → 類型「網頁應用程式」
 *    執行身分：我　／　誰可以存取：所有人 → 部署
 * 4. 函式選 showLinks → 執行 → 到「執行記錄」複製前台、後台連結
 *    （如果連結結尾是 /dev，請改用部署視窗顯示的「網頁應用程式網址」，結尾是 /exec，
 *      後台就是：網址 + ?page=admin&key=後台密碼）
 *
 * 之後修改程式：儲存後到「部署 → 管理部署作業 → ✏️ 編輯 → 版本選『新版本』→ 部署」，網址不會變
 */

var STATUSES = ['新需求', '已聯絡', '已報價', '已匯款', '製作中', '已寄出', '已完成', '未成交'];
var PAID = ['已匯款', '製作中', '已寄出', '已完成'];
var HEADERS = ['訂單編號', '建立時間', '月份', '狀態', '報價金額', '稱呼', 'IG', '聯絡方式', 'LINE／Email',
  '主要商品', '商品細節', '取貨方式', '收件人', '手機', '取貨門市', '參考照片', '作品分享', '從哪裡知道', '其他', '備註'];

// ---------- 第一次設定 ----------
function setup() {
  var props = PropertiesService.getScriptProperties();
  if (props.getProperty('SHEET_ID')) {
    Logger.log('已經設定過了，不用再執行。要看連結請執行 showLinks。');
    return;
  }
  var ss = SpreadsheetApp.create('Feiyen 訂單管理');
  var sh = ss.getSheets()[0].setName('訂單');
  sh.getRange(1, 1, 1, HEADERS.length).setValues([HEADERS]).setFontWeight('bold').setBackground('#f3e6d8');
  sh.setFrozenRows(1);
  sh.getRange('D2:D').setDataValidation(SpreadsheetApp.newDataValidation().requireValueInList(STATUSES, true).build());
  sh.getRange('E2:E').setNumberFormat('#,##0');
  props.setProperty('SHEET_ID', ss.getId());
  props.setProperty('ADMIN_KEY', Utilities.getUuid().replace(/-/g, '').slice(0, 12));
  Logger.log('✅ 設定完成！訂單試算表：' + ss.getUrl());
  Logger.log('下一步：部署成網頁應用程式，再執行 showLinks。');
}

function showLinks() {
  var url = ScriptApp.getService().getUrl();
  var key = PropertiesService.getScriptProperties().getProperty('ADMIN_KEY');
  if (!url) { Logger.log('還沒部署，請先「部署 → 新增部署作業 → 網頁應用程式」。'); return; }
  Logger.log('後台密碼（key）：' + key);
  Logger.log('前台（給客人）：' + url);
  Logger.log('後台（只給自己，不要外流）：' + url + '?page=admin&key=' + key);
  Logger.log('訂單試算表：' + SpreadsheetApp.openById(getProp('SHEET_ID')).getUrl());
}

function getProp(k) { return PropertiesService.getScriptProperties().getProperty(k); }
function sheet() { return SpreadsheetApp.openById(getProp('SHEET_ID')).getSheetByName('訂單'); }
function checkKey(key) { if (!key || key !== getProp('ADMIN_KEY')) throw new Error('沒有權限'); }

// ---------- 網頁 ----------
function doGet(e) {
  var p = (e && e.parameter) || {};
  var out;
  if (p.page === 'admin') {
    if (p.key !== getProp('ADMIN_KEY')) {
      return HtmlService.createHtmlOutput('<p style="font-family:sans-serif;padding:24px">沒有權限</p>');
    }
    out = HtmlService.createHtmlOutput(ADMIN_HTML.replace('__KEY__', p.key)).setTitle('Feiyen 訂單後台');
  } else {
    out = HtmlService.createHtmlOutput(FORM_HTML).setTitle('Feiyen 客製需求表');
  }
  return out.addMetaTag('viewport', 'width=device-width, initial-scale=1');
}

// ---------- 前台：客人送出 ----------
function submitOrder(d) {
  if (!d || d.website) return { ok: true }; // 機器人陷阱欄位
  var s = function (v, n) { return String(v == null ? '' : v).slice(0, n || 500); };
  if (!s(d.name).trim() || !s(d.ig).trim() || !s(d.main).trim()) throw new Error('必填欄位沒有填');
  var lock = LockService.getScriptLock();
  lock.waitLock(20000);
  try {
    var sh = sheet();
    var now = new Date();
    var month = Utilities.formatDate(now, 'Asia/Taipei', 'yyyy-MM');
    var id = 'F' + Utilities.formatDate(now, 'Asia/Taipei', 'yyMMdd') + '-' + ('00' + sh.getLastRow()).slice(-3);
    sh.appendRow([id, now, "'" + month, '新需求', '', s(d.name, 50), s(d.ig, 60), s(d.contact, 20), s(d.lineEmail, 100),
      s(d.main, 50), s(d.details, 2000), s(d.pickup, 20), s(d.rcvName, 50), "'" + s(d.rcvPhone, 20), s(d.rcvStore, 100),
      s(d.photo, 50), s(d.share, 50), s(d.source, 30), s(d.note, 1000), '']);
  } finally {
    lock.releaseLock();
  }
  try {
    var url = ScriptApp.getService().getUrl();
    MailApp.sendEmail(Session.getEffectiveUser().getEmail(),
      '【新需求】' + s(d.name, 50) + '｜' + s(d.main, 50),
      '收到一筆新的客製需求 🎨\n\n' + s(d.summary, 4000) +
      '\n\n——\n訂單編號：' + id + '\n後台：' + url + '?page=admin&key=' + getProp('ADMIN_KEY') +
      '\n記得 3 天內回覆客人喔！');
  } catch (err) { /* 寄信失敗不影響訂單存檔 */ }
  return { ok: true, id: id };
}

// ---------- 後台 ----------
function readOrders() {
  var sh = sheet();
  var n = sh.getLastRow() - 1;
  if (n < 1) return [];
  var rows = sh.getRange(2, 1, n, HEADERS.length).getValues();
  return rows.map(function (r, i) {
    var o = { row: i + 2 };
    HEADERS.forEach(function (h, j) { o[h] = r[j]; });
    o['建立時間'] = r[1] instanceof Date ? Utilities.formatDate(r[1], 'Asia/Taipei', 'yyyy-MM-dd HH:mm') : String(r[1]);
    o['月份'] = String(r[2]);
    o['報價金額'] = Number(r[4]) || 0;
    return o;
  });
}

function getDashboard(key, month) {
  checkKey(key);
  var all = readOrders();
  var months = {};
  all.forEach(function (o) { months[o['月份']] = true; });
  var thisMonth = Utilities.formatDate(new Date(), 'Asia/Taipei', 'yyyy-MM');
  months[thisMonth] = true;
  var monthList = Object.keys(months).sort().reverse();
  month = month || thisMonth;

  var list = all.filter(function (o) { return o['月份'] === month; });
  var paid = list.filter(function (o) { return PAID.indexOf(o['狀態']) >= 0; });
  var sum = function (a) { return a.reduce(function (t, o) { return t + o['報價金額']; }, 0); };
  var group = function (arr, field) {
    var m = {};
    arr.forEach(function (o) {
      var k = o[field] || '（未填）';
      m[k] = m[k] || { name: k, count: 0, paid: 0, revenue: 0 };
      m[k].count++;
      if (PAID.indexOf(o['狀態']) >= 0) { m[k].paid++; m[k].revenue += o['報價金額']; }
    });
    return Object.keys(m).map(function (k) { return m[k]; }).sort(function (a, b) { return b.count - a.count; });
  };
  var statusCount = {};
  STATUSES.forEach(function (st) { statusCount[st] = 0; });
  list.forEach(function (o) { statusCount[o['狀態']] = (statusCount[o['狀態']] || 0) + 1; });

  var trend = monthList.slice(0, 6).map(function (m) {
    var l = all.filter(function (o) { return o['月份'] === m; });
    var p = l.filter(function (o) { return PAID.indexOf(o['狀態']) >= 0; });
    return { month: m, count: l.length, paid: p.length, revenue: sum(p) };
  }).reverse();

  var todo = all.filter(function (o) { return ['新需求', '已聯絡', '已報價', '已匯款', '製作中'].indexOf(o['狀態']) >= 0; });

  return {
    month: month, months: monthList, statuses: STATUSES,
    total: list.length, paidCount: paid.length, revenue: sum(paid),
    conversion: list.length ? Math.round(paid.length / list.length * 100) : 0,
    statusCount: statusCount, byProduct: group(list, '主要商品'), bySource: group(list, '從哪裡知道'),
    trend: trend, todoCount: todo.length,
    orders: list.reverse()
  };
}

function updateOrder(key, id, status, amount, note) {
  checkKey(key);
  if (STATUSES.indexOf(status) < 0) throw new Error('狀態不正確');
  var sh = sheet();
  var ids = sh.getRange(2, 1, Math.max(sh.getLastRow() - 1, 1), 1).getValues();
  for (var i = 0; i < ids.length; i++) {
    if (ids[i][0] === id) {
      var r = i + 2;
      sh.getRange(r, 4).setValue(status);
      sh.getRange(r, 5).setValue(amount === '' || amount == null ? '' : Number(amount) || 0);
      sh.getRange(r, 20).setValue(String(note || '').slice(0, 1000));
      return { ok: true };
    }
  }
  throw new Error('找不到訂單 ' + id);
}

// ================= 前台網頁 =================
var FORM_HTML = String.raw`<!doctype html><html><head><meta charset="utf-8">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;600&family=Noto+Serif+TC:wght@500;600&family=Noto+Sans+TC:wght@300;400;500&display=swap">
<style>
:root{--bg:#f6efe7;--paper:#fffaf5;--ink:#3b2a22;--muted:#7d6a5e;--line:#e6d7c9;--accent:#c9566b;--accent-soft:#f7dfe2;--ok:#3f7d5a;--error:#b3261e;
--display:"Noto Serif TC","Songti TC","PMingLiU",serif;--latin:"Cormorant Garamond","Noto Serif TC",serif;--body:"Noto Sans TC","PingFang TC","Microsoft JhengHei",sans-serif}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font-family:var(--body);font-weight:400;font-size:15.5px;line-height:1.85;letter-spacing:.03em;padding:24px 16px 64px;-webkit-font-smoothing:antialiased}
[hidden]{display:none!important}
.wrap{max-width:640px;margin:0 auto;display:grid;gap:20px}
.brand{font-family:var(--latin);font-weight:600;font-size:17px;letter-spacing:.28em;color:var(--accent)}
h1{font-family:var(--display);font-weight:600;font-size:clamp(26px,6.5vw,34px);letter-spacing:.12em;line-height:1.4;margin:6px 0 8px}
.lead{color:var(--muted);font-weight:300;margin:0}
.notice{background:var(--accent-soft);border-radius:10px;padding:12px 14px;font-size:14px;margin:8px 0 0}
section{background:var(--paper);border:1px solid var(--line);border-radius:14px;padding:20px 18px;display:grid;gap:18px}
section h2{font-family:var(--display);font-weight:600;font-size:19px;letter-spacing:.1em;margin:0;padding-bottom:10px;border-bottom:1px solid var(--line)}
section h2 small{font-family:var(--body);font-size:12px;color:var(--muted);font-weight:400;letter-spacing:.05em;margin-left:10px}
.desc{white-space:pre-line;font-size:13.5px;font-weight:300;color:var(--muted);background:var(--bg);border-radius:10px;padding:10px 12px;margin:0}
.q{display:grid;gap:8px;border:0;margin:0;padding:0;min-width:0}
.q>label:first-child,.q legend{font-weight:500;font-size:15px;letter-spacing:.05em;padding:0}
.req::after{content:" *";color:var(--accent)}
.hint{font-size:13px;color:var(--muted);margin:0}
input[type=text],input[type=tel],input[type=number],textarea,select{width:100%;font:inherit;color:var(--ink);background:var(--bg);border:1px solid var(--line);border-radius:10px;padding:10px 12px}
textarea{min-height:84px}
.opts{display:flex;flex-wrap:wrap;gap:8px}
.opt{position:relative}
.opt input{position:absolute;opacity:0;inset:0;margin:0}
.opt span{display:inline-block;border:1px solid var(--line);background:var(--bg);border-radius:999px;padding:6px 15px;font-size:14px;letter-spacing:.04em;cursor:pointer}
.opt input:checked+span{background:var(--accent);border-color:var(--accent);color:#fff}
.opt input:focus-visible+span{outline:2px solid var(--accent);outline-offset:2px}
.err{color:var(--error);font-size:13px;margin:0}
.q.invalid input,.q.invalid select{border-color:var(--error)}
button{font:inherit;font-weight:500;letter-spacing:.2em;border:0;border-radius:12px;padding:15px 18px;cursor:pointer;background:var(--accent);color:#fff;font-size:16px;width:100%}
button:disabled{opacity:.6}
.guide{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px}
.guide img{width:100%;border-radius:10px;border:1px solid var(--line)}
.done{text-align:center}
.done .big{font-family:var(--display);font-weight:600;font-size:26px;letter-spacing:.1em;margin:0}
a{color:var(--accent)}
.hp{position:absolute;left:-9999px}
footer{text-align:center;color:var(--muted);font-family:var(--latin);font-size:14px;letter-spacing:.15em}
</style></head><body>
<div class="wrap">
<header>
  <div class="brand">FEIYEN 手繪 ・ @fei.ru.yen</div>
  <h1>客製需求調查表</h1>
  <p class="lead">先讓我了解你想做的內容，送出後我會在 3 天內私訊你討論細節並報價。</p>
  <p class="notice">這份是「需求調查」，填完不代表下單成功。確認報價並全額匯款後才會排單喔！</p>
</header>

<form id="form" novalidate>
<input type="text" name="website" id="website" class="hp" tabindex="-1" autocomplete="off" aria-hidden="true">

<section>
  <h2>基本資料</h2>
  <div class="q" data-req><label class="req" for="name">怎麼稱呼你？</label><input type="text" id="name"></div>
  <div class="q" data-req><label class="req" for="ig">你的 IG 帳號</label><input type="text" id="ig" placeholder="@your_ig"><p class="hint">我會用 IG 私訊聯絡你</p></div>
  <fieldset class="q" data-req data-group="contact"><legend class="req">偏好的聯絡方式</legend><div class="opts" data-opts="IG 私訊|LINE|Email" data-type="radio"></div></fieldset>
  <div class="q" id="q-lineEmail" hidden><label for="lineEmail">LINE ID 或 Email</label><input type="text" id="lineEmail"></div>
</section>

<section>
  <h2>想做的商品</h2>
  <fieldset class="q" data-req data-group="main"><legend class="req">這次主要想做哪一種商品？</legend><div class="opts" data-opts="無臉畫圖檔|無框畫|鑰匙圈|生日立牌／插旗|紅包袋／春聯|手機桌布|還不確定，想先聊聊" data-type="radio"></div></fieldset>
</section>

<section data-for="無臉畫圖檔" hidden>
  <h2>無臉畫圖檔<small>350 元起／1 位</small></h2>
  <fieldset class="q" data-req data-group="faceUse"><legend class="req">圖檔的用途</legend><div class="opts" data-opts="手機桌布|社群大頭貼|送禮|印成實體商品|婚禮或活動|其他" data-type="checkbox"></div></fieldset>
  <fieldset class="q" data-group="faceRatio"><legend>需要的尺寸或比例</legend><div class="opts" data-opts="正方形 1:1|直式 3:4|手機 9:16|橫式|請幫我建議" data-type="radio"></div></fieldset>
  <fieldset class="q" data-req data-group="faceUsage"><legend class="req">是否會用於商業販售？</legend><div class="opts" data-opts="個人使用|商業使用" data-type="radio"></div><p class="hint">商業使用（例如印成商品販售）另有授權費</p></fieldset>
  <fieldset class="q" data-group="faceAddon"><legend>想額外加購嗎？</legend><div class="opts" data-opts="無框畫 20×20 cm（$699）|鑰匙圈 5×5 cm（$450）" data-type="checkbox"></div><p class="hint">加購商品會用同一張無臉畫製作</p></fieldset>
</section>

<section data-for="無框畫" hidden>
  <h2>無框畫<small>20 × 20 cm・699 元</small></h2>
  <fieldset class="q" data-group="frameFile"><legend>是否也需要電子圖檔？</legend><div class="opts" data-opts="需要|不需要" data-type="radio"></div></fieldset>
</section>

<section data-for="鑰匙圈" hidden>
  <h2>鑰匙圈<small>5 × 5 cm 皮革・450 元</small></h2>
  <div class="q" data-req><label class="req" for="keyQty">預計數量</label><input type="number" id="keyQty" min="1" inputmode="numeric"></div>
  <fieldset class="q" data-group="keySide"><legend>單面或雙面</legend><div class="opts" data-opts="單面|雙面|不確定" data-type="radio"></div></fieldset>
</section>

<section data-for="生日立牌／插旗" hidden>
  <h2>生日立牌／插旗</h2>
  <p class="desc">🎂 生日商品請在生日或活動前 30 天下單
・生日立牌：60 × 60 cm，1,000 元（宅配）
・生日蛋糕插旗：1 支 90 元，訂單滿 200 元才出貨（3 支起）
・點心插旗：一組 12 支 200 元</p>
  <fieldset class="q" data-req data-group="bdayType"><legend class="req">要做哪一種？</legend><div class="opts" data-opts="生日立牌|生日蛋糕插旗|點心插旗" data-type="checkbox"></div></fieldset>
  <div class="q" data-req><label class="req" for="bdayName">壽星的名字（放在畫面上的寫法）</label><input type="text" id="bdayName"></div>
  <div class="q"><label for="bdayAge">幾歲生日？</label><input type="text" id="bdayAge"></div>
  <fieldset class="q" id="q-stand" data-req data-group="standStyle" hidden><legend class="req">生日立牌款式</legend><div class="opts" data-opts="A|B|C|D" data-type="radio"></div></fieldset>
  <fieldset class="q" id="q-cake" data-req data-group="cakeStyle" hidden><legend class="req">生日蛋糕插旗款式（共 17 款）</legend><div class="opts" data-opts="A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q" data-type="radio"></div></fieldset>
  <div class="q" id="q-cakeQty" data-req hidden><label class="req" for="cakeQty">生日蛋糕插旗支數</label><input type="number" id="cakeQty" min="3" inputmode="numeric"><p class="hint">3 支起</p></div>
  <fieldset class="q" id="q-snack" data-req data-group="snackStyle" hidden><legend class="req">點心插旗生日帽款式（共 6 款）</legend><div class="opts" data-opts="A|B|C|D|E|F" data-type="radio"></div></fieldset>
  <div class="q" id="q-snackQty" data-req hidden><label class="req" for="snackQty">點心插旗組數</label><input type="number" id="snackQty" min="1" inputmode="numeric"><p class="hint">一組 12 支</p></div>
</section>

<section data-for="紅包袋／春聯" hidden>
  <h2>紅包袋／春聯</h2>
  <p class="desc">🧧 2027 羊年春節商品 12/31 截單
・紅包袋：1 款 3 個 180 元
・春聯：1 款 2 張 150 元
・基本款每張畫面 1 位，加 1 位人物或寵物 +20 元，最多 4 位
・同款加印一張 +40 元</p>
  <div class="q" data-req><label class="req" for="redQty">紅包袋訂購數量</label><select id="redQty"><option value="">請選擇</option><option>不需要</option><option>1 款</option><option>2 款</option><option>3 款</option><option>4 款</option><option>5 款</option></select></div>
  <div class="q" data-req><label class="req" for="coupletQty">春聯訂購數量</label><select id="coupletQty"><option value="">請選擇</option><option>不需要</option><option>1 款</option><option>2 款</option><option>3 款</option><option>4 款</option><option>5 款</option></select></div>
  <fieldset class="q" data-req data-group="springPeople"><legend class="req">每張畫面要放幾位？（人物＋寵物）</legend><div class="opts" data-opts="1 位（基本款）|+1 位（$20）|+2 位（$40）|+3 位（$60）" data-type="radio"></div></fieldset>
</section>

<section data-for="手機桌布" hidden>
  <h2>手機桌布<small>一組 250 元</small></h2>
  <p class="desc">📱 一組包含鎖屏＋桌布共 2 張，1290 × 2796 px，電子檔交付
定期推出新款式，請參考 IG 最新公告</p>
  <div class="q" data-req><label class="req" for="wallStyle">想要的款式</label><input type="text" id="wallStyle" placeholder="填 IG 公告的款式代號，可填多款"></div>
</section>

<section id="s-ship" hidden>
  <h2>取貨與收件人資訊</h2>
  <p class="desc">運費由買方支付：超商店到店 60 元。生日立牌為宅配（80 元），我會再和你確認地址。</p>
  <fieldset class="q" data-req data-group="pickup"><legend class="req">取貨方式</legend><div class="opts" data-opts="7-11 店到店|全家店到店" data-type="radio"></div></fieldset>
  <div class="q" data-req><label class="req" for="rcvName">收件人姓名</label><input type="text" id="rcvName"></div>
  <div class="q" data-req data-phone><label class="req" for="rcvPhone">收件人手機</label><input type="tel" id="rcvPhone" inputmode="numeric" placeholder="09xxxxxxxx"></div>
  <div class="q" data-req><label class="req" for="rcvStore">取貨門市</label><input type="text" id="rcvStore" placeholder="例如：7-11 ○○門市（店號 123456）"></div>
</section>

<section>
  <h2>參考照片</h2>
  <p class="desc">📸 如何挑照片：光線明亮清楚、頭臉清楚無陰影、臉部正面無遮擋、近拍人物
🚫 NG 照片：人物太小、美肌或調色、模糊晃動、小圖放大、頭臉被切到、異物遮臉
每位人物或寵物都需要照片。照片只用於這次製作，不會公開。</p>
  <fieldset class="q" data-req data-group="photo"><legend class="req">參考照片怎麼提供？</legend><div class="opts" data-opts="送出後用 IG 私訊傳|用 LINE 傳|還沒準備好" data-type="radio"></div></fieldset>
</section>

<section>
  <h2>確認事項</h2>
  <fieldset class="q" data-req data-all data-group="agree"><legend class="req">我已了解</legend><div class="opts" data-opts="填表不代表下單成功，全額匯款後才排單|已閱讀 IG 精選「貼心提醒」" data-type="checkbox"></div></fieldset>
  <fieldset class="q" data-req data-group="share"><legend class="req">完成的作品可以分享在 IG 嗎？</legend><div class="opts" data-opts="可以|可以，但請遮住臉或名字|不可以" data-type="radio"></div></fieldset>
  <fieldset class="q" data-group="source"><legend>你是怎麼知道 Feiyen 手繪的？</legend><div class="opts" data-opts="IG 探索|朋友推薦|看到朋友的作品|回頭客|其他" data-type="radio"></div></fieldset>
  <div class="q"><label for="note">其他想說的話</label><textarea id="note"></textarea></div>
</section>

<p class="err" id="formErr" hidden>還有必填的題目沒填，請看標示紅字的地方。</p>
<button type="submit" id="submitBtn">送出需求</button>
</form>

<section id="done" class="done" hidden>
  <p class="big">收到你的需求了 💛</p>
  <p>我會在 3 天內私訊你。<br>參考照片可以先傳到 IG <b>@fei.ru.yen</b></p>
  <p><a href="https://ig.me/m/fei.ru.yen" target="_blank" rel="noopener">開啟 IG 私訊</a></p>
</section>
<footer>Feiyen 手繪 ・ IG @fei.ru.yen</footer>
</div>

<script>
(function(){
  var $=function(s){return document.querySelector(s)}, $$=function(s){return Array.prototype.slice.call(document.querySelectorAll(s))};
  var form=$('#form');
  $$('[data-opts]').forEach(function(box){
    var name=box.parentNode.getAttribute('data-group'), type=box.getAttribute('data-type');
    box.getAttribute('data-opts').split('|').forEach(function(v,i){
      var lab=document.createElement('label');lab.className='opt';
      var inp=document.createElement('input');inp.type=type;inp.name=name;inp.value=v;inp.id=name+'-'+i;
      var sp=document.createElement('span');sp.textContent=v;
      lab.appendChild(inp);lab.appendChild(sp);box.appendChild(lab);
    });
  });
  function val(n){var e=document.querySelector('input[name="'+n+'"]:checked');return e?e.value:''}
  function vals(n){return $$('input[name="'+n+'"]:checked').map(function(e){return e.value})}
  function v(id){return ($('#'+id).value||'').trim()}
  function hide(sel,h){var e=$(sel);if(e)e.hidden=h}
  function needsShip(){
    var m=val('main');
    if(m==='紅包袋／春聯')return !(v('redQty')==='不需要'&&v('coupletQty')==='不需要');
    if(['無框畫','鑰匙圈','生日立牌／插旗'].indexOf(m)>=0)return true;
    return m==='無臉畫圖檔'&&vals('faceAddon').length>0;
  }
  function update(){
    var m=val('main');
    $$('section[data-for]').forEach(function(s){s.hidden=s.getAttribute('data-for')!==m});
    hide('#q-lineEmail',['LINE','Email'].indexOf(val('contact'))<0);
    var t=vals('bdayType');
    hide('#q-stand',t.indexOf('生日立牌')<0);
    hide('#q-cake',t.indexOf('生日蛋糕插旗')<0);hide('#q-cakeQty',t.indexOf('生日蛋糕插旗')<0);
    hide('#q-snack',t.indexOf('點心插旗')<0);hide('#q-snackQty',t.indexOf('點心插旗')<0);
    hide('#s-ship',!needsShip());
  }
  function visible(el){return !el.closest('[hidden]')}
  function mark(q,msg){q.classList.add('invalid');var p=document.createElement('p');p.className='err';p.textContent=msg;q.appendChild(p)}
  function validate(){
    var ok=true;
    $$('.q').forEach(function(q){
      q.classList.remove('invalid');var o=q.querySelector('.err');if(o)o.remove();
      if(!visible(q)||!q.hasAttribute('data-req'))return;
      var g=q.getAttribute('data-group'),msg='';
      if(g){var n=vals(g).length,all=$$('input[name="'+g+'"]').length;
        if(q.hasAttribute('data-all')?n<all:n===0)msg=q.hasAttribute('data-all')?'請勾選全部項目':'請選擇一項';}
      else{var inp=q.querySelector('input,select,textarea'),x=(inp.value||'').trim();
        if(!x)msg='這題必填';
        else if(q.hasAttribute('data-phone')&&!/^09[0-9]{8}$/.test(x.replace(/[ -]/g,'')))msg='請填 09 開頭的 10 碼手機號碼';
        else if(inp.id==='cakeQty'&&+x<3)msg='生日蛋糕插旗 3 支起';
        else if(inp.type==='number'&&+x<1)msg='請填 1 以上的數字';}
      if(msg){ok=false;mark(q,msg)}
    });
    if(val('main')==='紅包袋／春聯'&&v('redQty')==='不需要'&&v('coupletQty')==='不需要'){ok=false;mark($('#coupletQty').closest('.q'),'紅包袋和春聯至少要選一種')}
    return ok;
  }
  function details(){
    var L=[],m=val('main');
    function add(k,x){if(x&&(!Array.isArray(x)||x.length))L.push(k+'：'+(Array.isArray(x)?x.join('、'):x))}
    if(m==='無臉畫圖檔'){add('用途',vals('faceUse'));add('尺寸比例',val('faceRatio'));add('使用方式',val('faceUsage'));add('加購',vals('faceAddon'))}
    if(m==='無框畫'){add('電子圖檔',val('frameFile'))}
    if(m==='鑰匙圈'){add('數量',v('keyQty'));add('單雙面',val('keySide'))}
    if(m==='生日立牌／插旗'){var t=vals('bdayType');add('品項',t);add('壽星名字',v('bdayName'));add('幾歲',v('bdayAge'));
      if(t.indexOf('生日立牌')>=0)add('立牌款式',val('standStyle'));
      if(t.indexOf('生日蛋糕插旗')>=0){add('蛋糕插旗款式',val('cakeStyle'));add('蛋糕插旗支數',v('cakeQty'))}
      if(t.indexOf('點心插旗')>=0){add('點心插旗款式',val('snackStyle'));add('點心插旗組數',v('snackQty'))}}
    if(m==='紅包袋／春聯'){add('紅包袋',v('redQty'));add('春聯',v('coupletQty'));add('每張畫面',val('springPeople'))}
    if(m==='手機桌布'){add('款式',v('wallStyle'))}
    return L;
  }
  form.addEventListener('input',update);form.addEventListener('change',update);
  form.addEventListener('submit',function(e){
    e.preventDefault();update();
    var ok=validate();$('#formErr').hidden=ok;
    if(!ok){var f=$('.q.invalid');if(f)f.scrollIntoView({behavior:'smooth',block:'center'});return}
    var ship=!$('#s-ship').hidden, det=details();
    var d={website:v('website'),name:v('name'),ig:v('ig'),contact:val('contact'),lineEmail:$('#q-lineEmail').hidden?'':v('lineEmail'),
      main:val('main'),details:det.join('\n'),pickup:ship?val('pickup'):'',rcvName:ship?v('rcvName'):'',rcvPhone:ship?v('rcvPhone'):'',rcvStore:ship?v('rcvStore'):'',
      photo:val('photo'),share:val('share'),source:val('source'),note:v('note')};
    d.summary=['稱呼：'+d.name,'IG：'+d.ig,'聯絡方式：'+d.contact+(d.lineEmail?'（'+d.lineEmail+'）':''),'','主要商品：'+d.main].concat(det)
      .concat(ship?['','取貨：'+d.pickup,'收件人：'+d.rcvName,'手機：'+d.rcvPhone,'門市：'+d.rcvStore]:[])
      .concat(['','參考照片：'+d.photo,'作品分享：'+d.share,'從哪裡知道：'+d.source,'其他：'+d.note]).join('\n');
    var btn=$('#submitBtn');btn.disabled=true;btn.textContent='送出中⋯';
    google.script.run.withSuccessHandler(function(){form.hidden=true;$('#done').hidden=false;window.scrollTo(0,0)})
      .withFailureHandler(function(){btn.disabled=false;btn.textContent='送出需求';$('#formErr').textContent='送出失敗，請稍後再試，或直接私訊 IG @fei.ru.yen';$('#formErr').hidden=false})
      .submitOrder(d);
  });
  update();
})();
</script></body></html>`;

// ================= 後台網頁 =================
var ADMIN_HTML = String.raw`<!doctype html><html><head><meta charset="utf-8">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=LXGW+WenKai+TC:wght@700&family=Noto+Sans+TC:wght@400;500;700&display=swap">
<style>
:root{--bg:#f6efe7;--paper:#fffaf5;--ink:#3b2a22;--muted:#7d6a5e;--line:#e6d7c9;--accent:#c9566b;--accent-soft:#f7dfe2;--ok:#3f7d5a;
--display:"LXGW WenKai TC","Kaiti TC","Noto Sans TC",serif;--body:"Noto Sans TC","PingFang TC",sans-serif}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font-family:var(--body);font-size:15px;line-height:1.6;padding:20px 16px 60px}
[hidden]{display:none!important}
.wrap{max-width:960px;margin:0 auto;display:grid;gap:18px}
header{display:flex;flex-wrap:wrap;gap:12px;align-items:end;justify-content:space-between}
h1{font-family:var(--display);font-size:30px;margin:0}
h2{font-family:var(--display);font-size:20px;margin:0 0 10px}
select,input,textarea{font:inherit;color:var(--ink);background:var(--paper);border:1px solid var(--line);border-radius:8px;padding:6px 10px}
.cards{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:12px}
.card{background:var(--paper);border:1px solid var(--line);border-radius:12px;padding:14px}
.card .k{font-size:13px;color:var(--muted)}
.card .v{font-size:26px;font-weight:700;font-variant-numeric:tabular-nums}
.panel{background:var(--paper);border:1px solid var(--line);border-radius:12px;padding:16px;min-width:0}
.two{display:grid;grid-template-columns:repeat(auto-fit,minmax(280px,1fr));gap:12px}
.tbl{overflow-x:auto}
table{width:100%;border-collapse:collapse;font-variant-numeric:tabular-nums}
th,td{text-align:left;padding:6px 8px;border-bottom:1px solid var(--line);white-space:nowrap}
th{font-size:13px;color:var(--muted);font-weight:500}
td.n,th.n{text-align:right}
.bar{height:8px;border-radius:4px;background:var(--accent);display:block;min-width:2px}
.chips{display:flex;flex-wrap:wrap;gap:6px}
.chip{border:1px solid var(--line);border-radius:999px;padding:3px 10px;font-size:13px;background:var(--paper);cursor:pointer}
.chip.on{background:var(--accent);border-color:var(--accent);color:#fff}
.order{border:1px solid var(--line);border-radius:12px;background:var(--paper);padding:14px;display:grid;gap:8px}
.order .top{display:flex;flex-wrap:wrap;gap:8px;justify-content:space-between;align-items:baseline}
.order .who{font-weight:700;font-size:16px}
.meta{font-size:13px;color:var(--muted)}
.st{font-size:12px;border-radius:999px;padding:2px 10px;background:var(--accent-soft)}
.st.paid{background:#dcefe3;color:var(--ok)}
pre{white-space:pre-wrap;margin:0;font-family:var(--body);font-size:14px;background:var(--bg);border-radius:8px;padding:8px 10px}
.edit{display:flex;flex-wrap:wrap;gap:8px;align-items:center}
.edit input[type=number]{width:120px}
.edit input[type=text]{flex:1;min-width:160px}
button{font:inherit;font-weight:700;border:0;border-radius:8px;padding:7px 14px;background:var(--accent);color:#fff;cursor:pointer}
.saved{color:var(--ok);font-size:13px}
.empty{color:var(--muted);text-align:center;padding:24px}
#loading{color:var(--muted)}
</style></head><body>
<div class="wrap">
<header><div><div class="meta">FEIYEN 手繪</div><h1>訂單後台</h1></div>
<label>月份 <select id="month"></select></label></header>
<p id="loading">讀取中⋯</p>
<div id="app" hidden>
  <div class="cards">
    <div class="card"><div class="k">本月需求數</div><div class="v" id="c-total">0</div></div>
    <div class="card"><div class="k">成交（已匯款以後）</div><div class="v" id="c-paid">0</div></div>
    <div class="card"><div class="k">本月營收</div><div class="v" id="c-rev">0</div></div>
    <div class="card"><div class="k">成交率</div><div class="v" id="c-conv">0%</div></div>
    <div class="card"><div class="k">待處理（全部月份）</div><div class="v" id="c-todo">0</div></div>
  </div>
  <div class="two" style="margin-top:12px">
    <div class="panel"><h2>各商品</h2><div class="tbl"><table id="t-product"></table></div></div>
    <div class="panel"><h2>客人從哪裡來</h2><div class="tbl"><table id="t-source"></table></div></div>
  </div>
  <div class="panel" style="margin-top:12px"><h2>近 6 個月</h2><div class="tbl"><table id="t-trend"></table></div></div>
  <div class="panel" style="margin-top:12px"><h2>本月訂單</h2>
    <div class="chips" id="filters" style="margin-bottom:10px"></div>
    <div id="orders" style="display:grid;gap:10px"></div>
  </div>
</div>
</div>
<script>
var KEY='__KEY__', DATA=null, FILTER='全部', PAID=['已匯款','製作中','已寄出','已完成'];
function $(s){return document.querySelector(s)}
function el(tag,cls,txt){var e=document.createElement(tag);if(cls)e.className=cls;if(txt!=null)e.textContent=txt;return e}
function money(n){return '$'+Number(n||0).toLocaleString()}
function load(month){
  $('#loading').hidden=false;
  google.script.run.withSuccessHandler(render).withFailureHandler(function(e){$('#loading').textContent='讀取失敗：'+e.message}).getDashboard(KEY,month||'');
}
function table(t,head,rows){
  t.innerHTML='';var tr=el('tr');head.forEach(function(h,i){var th=el('th',i?'n':'',h);tr.appendChild(th)});t.appendChild(tr);
  if(!rows.length){var r=el('tr'),td=el('td','', '本月還沒有資料');td.colSpan=head.length;r.appendChild(td);t.appendChild(r);return}
  rows.forEach(function(cells){var r=el('tr');cells.forEach(function(c,i){var td=el('td',i?'n':'');if(c instanceof Node)td.appendChild(c);else td.textContent=c;r.appendChild(td)});t.appendChild(r)});
}
function render(d){
  DATA=d;$('#loading').hidden=true;$('#app').hidden=false;
  var sel=$('#month');sel.innerHTML='';d.months.forEach(function(m){var o=el('option','',m);o.value=m;if(m===d.month)o.selected=true;sel.appendChild(o)});
  $('#c-total').textContent=d.total;$('#c-paid').textContent=d.paidCount;$('#c-rev').textContent=money(d.revenue);
  $('#c-conv').textContent=d.conversion+'%';$('#c-todo').textContent=d.todoCount;
  table($('#t-product'),['商品','需求','成交','營收'],d.byProduct.map(function(p){return [p.name,p.count,p.paid,money(p.revenue)]}));
  table($('#t-source'),['來源','需求','成交'],d.bySource.map(function(p){return [p.name,p.count,p.paid]}));
  var max=Math.max.apply(null,d.trend.map(function(t){return t.revenue}).concat([1]));
  table($('#t-trend'),['月份','需求','成交','營收',''],d.trend.map(function(t){var b=el('span','bar');b.style.width=Math.round(t.revenue/max*120)+'px';return [t.month,t.count,t.paid,money(t.revenue),b]}));
  var f=$('#filters');f.innerHTML='';['全部'].concat(d.statuses).forEach(function(s){
    var n=s==='全部'?d.total:(d.statusCount[s]||0);var c=el('button','chip'+(s===FILTER?' on':''),s+' '+n);c.type='button';
    c.onclick=function(){FILTER=s;render(DATA)};f.appendChild(c)});
  var box=$('#orders');box.innerHTML='';
  var list=d.orders.filter(function(o){return FILTER==='全部'||o['狀態']===FILTER});
  if(!list.length){box.appendChild(el('p','empty','沒有符合的訂單'));return}
  list.forEach(function(o){
    var card=el('div','order');var top=el('div','top');
    var left=el('div');left.appendChild(el('span','who',o['稱呼']+'　'));left.appendChild(el('span','meta',o['IG']));
    top.appendChild(left);top.appendChild(el('span','st'+(PAID.indexOf(o['狀態'])>=0?' paid':''),o['狀態']));card.appendChild(top);
    card.appendChild(el('div','meta',o['訂單編號']+'・'+o['建立時間']+'・'+o['主要商品']+'・聯絡：'+o['聯絡方式']+(o['LINE／Email']?'（'+o['LINE／Email']+'）':'')));
    var info=[o['商品細節']];
    if(o['取貨方式'])info.push('取貨：'+o['取貨方式']+'｜'+o['收件人']+'｜'+o['手機']+'｜'+o['取貨門市']);
    info.push('照片：'+o['參考照片']+'｜分享：'+o['作品分享']+'｜來源：'+(o['從哪裡知道']||'—'));
    if(o['其他'])info.push('其他：'+o['其他']);
    card.appendChild(el('pre','',info.join('\n')));
    var ed=el('div','edit');var s=el('select');d.statuses.forEach(function(x){var op=el('option','',x);if(x===o['狀態'])op.selected=true;s.appendChild(op)});
    var a=el('input');a.type='number';a.placeholder='報價金額';a.value=o['報價金額']||'';
    var nt=el('input');nt.type='text';nt.placeholder='備註';nt.value=o['備註']||'';
    var b=el('button','', '儲存');b.type='button';var ok=el('span','saved','');
    b.onclick=function(){b.disabled=true;ok.textContent='儲存中⋯';
      google.script.run.withSuccessHandler(function(){ok.textContent='✓ 已儲存';b.disabled=false;load(DATA.month)})
        .withFailureHandler(function(e){ok.textContent='儲存失敗：'+e.message;b.disabled=false}).updateOrder(KEY,o['訂單編號'],s.value,a.value,nt.value)};
    ed.appendChild(s);ed.appendChild(a);ed.appendChild(nt);ed.appendChild(b);ed.appendChild(ok);card.appendChild(ed);
    box.appendChild(card);
  });
}
$('#month').addEventListener('change',function(){FILTER='全部';load(this.value)});
load('');
</script></body></html>`;
