/**
 * Feiyen 手繪｜一鍵建立「客製需求調查表」Google 表單
 *
 * 使用方式：
 * 1. 到 https://script.google.com 新增專案，刪掉原本的內容，貼上這整份
 * 2. 上方函式選單選「createFeiyenForm」，按「執行」
 * 3. 依畫面完成 Google 授權
 * 4. 到「執行紀錄」複製：表單填寫連結、表單編輯連結、回覆試算表連結
 *
 * 會自動完成：
 * - 建立表單（33 題＋插旗數量 2 題，依商品自動跳題）
 * - 建立回覆試算表
 * - 客人送出後，寄一封「含完整答案」的 Email 給你
 */

// 收通知信的 Email。留空會寄到你目前登入的 Google 帳號
var NOTIFY_EMAIL = '';

function createFeiyenForm() {
  var form = FormApp.create('Feiyen 手繪｜客製需求調查表');
  form.setDescription(
    '嗨嗨～謝謝你對 Feiyen 手繪有興趣 🥰\n\n' +
    '這份表單是「需求調查」，幫助我先了解你想做的內容。\n' +
    '填完後我會私訊和你討論細節並報價，確認報價並全額匯款後才算正式下單喔！\n\n' +
    '我是兼職畫家，會在 3 天內回覆，請見諒～\n' +
    '填寫前建議先看 IG 精選「貼心提醒」和「客製化商品」 ✨'
  );
  form.setCollectEmail(false);
  form.setLimitOneResponsePerUser(false);
  form.setProgressBar(true);
  form.setConfirmationMessage(
    '收到你的需求了，謝謝你 💛\n' +
    '我會在 3 天內用你選的方式聯絡你，\n' +
    '如果有參考照片，可以先私訊 IG @fei.ru.yen 給我喔！'
  );

  // ---------- 區段 1：基本資料 ----------
  form.addTextItem().setTitle('怎麼稱呼你？').setRequired(true);
  form.addTextItem().setTitle('你的 IG 帳號')
    .setHelpText('我會用 IG 私訊聯絡你，請填 @ 開頭的帳號').setRequired(true);
  form.addMultipleChoiceItem().setTitle('偏好的聯絡方式')
    .setChoiceValues(['IG 私訊', 'LINE', 'Email']).setRequired(true);
  form.addTextItem().setTitle('LINE ID 或 Email').setHelpText('選 LINE 或 Email 的人再填');

  // ---------- 區段 2：想做的商品 ----------
  form.addPageBreakItem().setTitle('想做的商品');
  var mainItem = form.addMultipleChoiceItem().setTitle('這次主要想做哪一種商品？').setRequired(true);
  form.addCheckboxItem().setTitle('有想一起加購的商品嗎？')
    .setChoiceValues(['無臉畫圖檔', '無框畫', '鑰匙圈', '生日立牌', '生日蛋糕插旗', '點心插旗', '紅包袋', '春聯', '手機桌布', '暫時不用']);

  // ---------- 區段 3A：無臉畫圖檔 ----------
  var pA = form.addPageBreakItem().setTitle('無臉畫圖檔').setHelpText('350 元起（含 1 位人物），加 1 位人物或寵物 +350 元');
  form.addCheckboxItem().setTitle('圖檔的用途')
    .setChoiceValues(['手機桌布', '社群大頭貼', '送禮', '印成實體商品', '婚禮或活動使用', '其他']).setRequired(true);
  form.addMultipleChoiceItem().setTitle('需要的尺寸或比例')
    .setChoiceValues(['正方形 1:1', '直式 3:4', '手機桌布 9:16', '橫式', '不確定，請幫我建議']);
  form.addMultipleChoiceItem().setTitle('是否會用於商業販售？')
    .setHelpText('商業使用（例如印成商品販售）另有授權費')
    .setChoiceValues(['個人使用', '商業使用']).setRequired(true);
  form.addCheckboxItem().setTitle('想額外加購嗎？')
    .setHelpText('加購商品會用同一張無臉畫製作')
    .setChoiceValues(['無框畫（20 × 20 cm，$699）', '鑰匙圈（5 × 5 cm，$450）', '暫時不用']);

  // ---------- 區段 3B：無框畫 ----------
  var pB = form.addPageBreakItem().setTitle('無框畫').setHelpText('無框畫為加購商品，尺寸 20 × 20 cm，699 元');
  form.addMultipleChoiceItem().setTitle('是否也需要電子圖檔？').setChoiceValues(['需要', '不需要']);

  // ---------- 區段 3C：鑰匙圈 ----------
  var pC = form.addPageBreakItem().setTitle('鑰匙圈').setHelpText('5 × 5 cm 皮革，450 元（含 1 位人物），加 1 位人物 +40 元');
  form.addTextItem().setTitle('預計數量').setHelpText('數量較多可另外討論優惠').setRequired(true)
    .setValidation(FormApp.createTextValidation().requireNumberGreaterThanOrEqualTo(1)
      .setHelpText('請填 1 以上的數字').build());
  form.addMultipleChoiceItem().setTitle('單面或雙面').setChoiceValues(['單面', '雙面', '不確定']);

  // ---------- 區段 3D：生日商品 ----------
  var pD = form.addPageBreakItem().setTitle('生日立牌／生日蛋糕插旗／點心插旗').setHelpText(
    '🎂 生日商品請在生日或活動前 30 天下單\n' +
    '・生日立牌：60 × 60 cm，1,000 元（宅配）\n' +
    '・生日蛋糕插旗：1 支 90 元，訂單滿 200 元才出貨（3 支起）\n' +
    '・點心插旗：一組 12 支 200 元\n' +
    '・插旗加 1 位人物 +40 元');
  form.addCheckboxItem().setTitle('要做哪一種？')
    .setChoiceValues(['生日立牌', '生日蛋糕插旗', '點心插旗']).setRequired(true);
  form.addTextItem().setTitle('壽星的名字（要放在畫面上的寫法）').setRequired(true);
  form.addTextItem().setTitle('幾歲生日？');
  form.addMultipleChoiceItem().setTitle('生日立牌款式').setHelpText('有選生日立牌的再填')
    .setChoiceValues(letters('ABCD'));
  form.addMultipleChoiceItem().setTitle('生日蛋糕插旗款式（共 17 款）').setHelpText('有選生日蛋糕插旗的再填')
    .setChoiceValues(letters('ABCDEFGHIJKLMNOPQ'));
  form.addTextItem().setTitle('生日蛋糕插旗支數').setHelpText('3 支起（滿 200 元出貨）')
    .setValidation(FormApp.createTextValidation().requireNumberGreaterThanOrEqualTo(3)
      .setHelpText('生日蛋糕插旗 3 支起').build());
  form.addMultipleChoiceItem().setTitle('點心插旗生日帽款式（共 6 款）').setHelpText('有選點心插旗的再填')
    .setChoiceValues(letters('ABCDEF'));
  form.addTextItem().setTitle('點心插旗組數').setHelpText('一組 12 支')
    .setValidation(FormApp.createTextValidation().requireNumberGreaterThanOrEqualTo(1)
      .setHelpText('請填 1 以上的數字').build());

  // ---------- 區段 3E：紅包袋／春聯 ----------
  var pE = form.addPageBreakItem().setTitle('紅包袋／春聯').setHelpText(
    '🧧 2027 羊年春節商品 12/31 截單，建議提早預約\n' +
    '・紅包袋：1 款 3 個 180 元\n' +
    '・春聯：1 款 2 張 150 元\n' +
    '・基本款每張畫面 1 位人物，加 1 位人物或寵物 +20 元，最多 4 位\n' +
    '・同款加印一張 +40 元');
  var qty = ['不需要', '1 款', '2 款', '3 款', '4 款', '5 款'];
  form.addMultipleChoiceItem().setTitle('紅包袋訂購數量').setHelpText('1 款 3 個').setChoiceValues(qty).setRequired(true);
  form.addMultipleChoiceItem().setTitle('春聯訂購數量').setHelpText('1 款 2 張').setChoiceValues(qty).setRequired(true);
  form.addMultipleChoiceItem().setTitle('每張畫面要放幾位？（人物＋寵物）')
    .setChoiceValues(['1 位（基本款）', '+1 位（$20）', '+2 位（$40）', '+3 位（$60）']).setRequired(true);

  // ---------- 區段 3F：手機桌布 ----------
  var pF = form.addPageBreakItem().setTitle('手機桌布').setHelpText(
    '📱 一組 250 元，包含鎖屏＋桌布共 2 張\n' +
    '・尺寸：1290 × 2796 px，300 dpi\n' +
    '・電子檔交付，不用寄送\n' +
    '・定期推出新款式，請參考 IG 最新公告');
  form.addTextItem().setTitle('想要的款式').setHelpText('填 IG 公告的款式代號，可填多款，每款一組 250 元').setRequired(true);

  // ---------- 區段 4：取貨與收件人資訊 ----------
  var pPickup = form.addPageBreakItem().setTitle('取貨與收件人資訊').setHelpText(
    '運費由買方支付：超商店到店 60 元。生日立牌為宅配（80 元），我會再和你確認地址。\n' +
    '只買無臉畫圖檔或手機桌布（電子檔）的話，這頁不用填。');
  form.addMultipleChoiceItem().setTitle('取貨方式').setChoiceValues(['7-11 店到店', '全家店到店']);
  form.addTextItem().setTitle('收件人姓名').setHelpText('請填真實姓名，取貨時需核對');
  form.addTextItem().setTitle('收件人手機').setHelpText('取貨通知簡訊會傳到這支手機')
    .setValidation(FormApp.createTextValidation().requireTextMatchesPattern('^09\\d{8}$')
      .setHelpText('請填 09 開頭的 10 碼手機號碼，不要有空格或「-」').build());
  form.addTextItem().setTitle('取貨門市').setHelpText('請填門市名稱或店號，例如「7-11 ○○門市（店號 123456）」');

  // ---------- 區段 5：參考照片 ----------
  form.addPageBreakItem().setTitle('參考照片').setHelpText(
    '📸 如何挑照片\n' +
    '・光線明亮清楚\n・頭、臉清楚、無陰影\n・臉部正面、無遮擋\n・近拍人物\n\n' +
    '🚫 NG 照片\n' +
    '・人物太小、美肌或調色、模糊晃動\n・小圖放大、頭臉被切到、異物遮臉\n\n' +
    '每位人物或寵物都需要照片，可以分開拍。照片只用於這次製作，不會公開或作其他用途。');
  form.addMultipleChoiceItem().setTitle('參考照片怎麼提供？')
    .setChoiceValues(['填完後用 IG 私訊傳給你', '用 LINE 傳給你', '還沒準備好']).setRequired(true);

  // ---------- 區段 6：確認事項 ----------
  form.addPageBreakItem().setTitle('確認事項');
  form.addCheckboxItem().setTitle('我已了解')
    .setChoiceValues(['填寫表單不代表下單成功，需確認報價並全額匯款後才會排單', '我已閱讀 IG 精選「貼心提醒」'])
    .setRequired(true)
    .setValidation(FormApp.createCheckboxValidation().requireSelectAtLeast(2).setHelpText('請勾選全部項目').build());
  form.addMultipleChoiceItem().setTitle('完成的作品可以分享在 IG 嗎？')
    .setChoiceValues(['可以', '可以，但請遮住臉或名字', '不可以']).setRequired(true);
  form.addMultipleChoiceItem().setTitle('你是怎麼知道 Feiyen 手繪的？')
    .setChoiceValues(['IG 探索', '朋友推薦', '看到朋友的作品', '回頭客', '其他']);
  form.addParagraphTextItem().setTitle('其他想說的話');

  // ---------- 跳題設定 ----------
  mainItem.setChoices([
    mainItem.createChoice('無臉畫圖檔', pA),
    mainItem.createChoice('無框畫', pB),
    mainItem.createChoice('鑰匙圈', pC),
    mainItem.createChoice('生日立牌、生日蛋糕插旗或點心插旗', pD),
    mainItem.createChoice('紅包袋或春聯', pE),
    mainItem.createChoice('手機桌布', pF),
    mainItem.createChoice('還不確定，想先聊聊', pPickup)
  ]);
  // 每個商品區段填完，直接跳到「取貨與收件人資訊」
  // （setGoToPage 設在「下一個區段」的分頁上，代表上一個區段結束後要去哪裡）
  [pB, pC, pD, pE, pF].forEach(function (p) { p.setGoToPage(pPickup); });

  // ---------- 回覆試算表 ----------
  var ss = SpreadsheetApp.create('Feiyen 需求調查表（回覆）');
  form.setDestination(FormApp.DestinationType.SPREADSHEET, ss.getId());

  // ---------- 送出後寄信通知 ----------
  ScriptApp.newTrigger('notifyOnSubmit').forForm(form).onFormSubmit().create();

  Logger.log('✅ 建立完成！');
  Logger.log('給客人的填寫連結：' + form.getPublishedUrl());
  Logger.log('表單編輯連結：' + form.getEditUrl());
  Logger.log('回覆試算表：' + ss.getUrl());
}

function letters(s) {
  return s.split('').map(function (c) { return c + ' 款'; });
}

// 客人送出表單時自動執行：寄一封含完整答案的 Email
function notifyOnSubmit(e) {
  var to = NOTIFY_EMAIL || Session.getEffectiveUser().getEmail();
  var items = e.response.getItemResponses();
  var name = '', main = '';
  var lines = items.map(function (r) {
    var title = r.getItem().getTitle();
    var ans = r.getResponse();
    if (Array.isArray(ans)) ans = ans.join('、');
    if (title === '怎麼稱呼你？') name = ans;
    if (title === '這次主要想做哪一種商品？') main = ans;
    return '■ ' + title + '\n' + ans;
  });
  var body =
    '收到一筆新的客製需求 🎨\n' +
    '填寫時間：' + Utilities.formatDate(e.response.getTimestamp(), 'Asia/Taipei', 'yyyy-MM-dd HH:mm') + '\n\n' +
    lines.join('\n\n') +
    '\n\n——\n記得 3 天內回覆客人喔！';
  MailApp.sendEmail(to, '【新需求】' + (name || '客人') + '｜' + (main || '未選商品'), body);
}
