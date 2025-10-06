## Hi there 👋
## note
## updated my java code

package com.autotest.pages.csrpage.csrsupport;

import java.time.Duration;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Set;

import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.WindowType;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;

import com.autotest.pages.CommonPage;

import submodule.com.autotest.commonpage.AbstractBasePage;

public class CSROneCallPass extends AbstractBasePage {

	// ===== 常數 =====
	private static final String POLICY_NO = "80241059";

	// Step1
	private String selectPolicyChangeTableMenuLocator = "//table[@width='100%' and contains(@class,'search-table')] //tr[2]/th[not(contains(@class,'pc_hide'))]";
	private By selectChangeItemOptLocator = By
			.xpath("//table[@width='100%' and @class='table table-bordered search-table' and @id='tran_list']");
	private String policyChangeCheckboxLocator = "//td[@class='pc_info']//div[@class='checkbox checkbox-inline checkbox-main']//label[@for='checkbox80241059']";
	private String policyLoanRadioButtonLocator = "//td[@class='pc_info']//label[@for='ewOption1']";
	private String stepOneNextButtonLocator = "//div[@class='btn_box_d']/a[@id='next']";

	// Step1 - 契約終止
	private String terminiateContractRadioButtonLocator = "//td[@class='pc_info']//label[@for='ewOption2']";

	// Step2（保單借款）
	private String policyChangeTableTitleLocator = "(//div[@id='divInputChg']//table[@width='100%' and @class='table table-bordered search-table']/tbody/tr)[1]/th[not(contains(@class,'pc_hide'))]";
	private String enterLoanContentTableTitleLocator = "//div[@id='divInputChg']//table[@width='100%' and @class='table table-bordered search-table' and @id='loanData2']/tbody/tr/th[not(contains(@class,'pc_hide'))]";

	// 下拉（bootstrap / 原生）
	private String loanTypeDropDownlistLocator = "//select[@class='selectpicker synSelect form-control']";
	private String loanReasonDropdownLocator = "(//select[@class='selectpicker synSelect select-resason'])[1]";

	// 付款方式
	private By useProvidedRadio = By.xpath("//div[@class='radio radio-inline radio-main form-group']//label[@for='TOaccount']");
	private By providedAccountSelect = By.id("TOaccountlist");

	// 收件人資訊
	private By chargeAddressRadio = By.xpath("//label[@for='cAddress']");
	private By chargeAddressInput = By.id("collectAddress");
	private By recipientNameSelect = By.id("clinetName");
	private By mailTypeSelect = By.id("mailType");

	// Step2 -> Step3（預覽）
	private By reviewButtonXPath = By.xpath("//a[@id='review' and normalize-space(text())='申請完成-預覽異動內容']");
	private By reviewButtonId    = By.id("review");

	// 暫存（保單借款）
	private List<String> stepTwoPolicyRow = new ArrayList<>();
	private List<String> stepTwoLoanRowHead5 = new ArrayList<>();
	private String selectedLoanTypeValue;
	private String selectedLoanTypeText;
	private String selectedLoanReasonValue;
	private String selectedLoanReasonText;
	private String inputLoanAmount;
	private String selectedProvidedAcctText;
	private String recipientNameText;
	private String mailTypeText;
	private String chargeAddressFull;
	private List<String> stepTwoDocs = new ArrayList<>();
	private List<String> stepTwoNotes = new ArrayList<>();

	// 契約終止 Step2
	private String terminateContractStep2ChangePolicyTableHeadersLocator = "(//div[@id='divInputChg']//table[@width='100%' and @class='table table-bordered search-table']/tbody/tr)[1]/th[not(contains(@class,'pc_hide'))]";
	private String cancellationReasonTableHeadersLocator = "(//div[@id='divInputChg']//table[@id='terminate2']/tbody/tr)[1]/th[not(contains(@class,'pc_hide'))]";

	private List<String> termStep2PolicyRow = new ArrayList<>();
	private String termSelectedReasonText;
	private String termSelectedProvidedAcctText;
	private String termRecipientNameText;
	private String termMailTypeText;
	private String termChargeAddressFull;
	private List<String> termStep2Docs = new ArrayList<>();
	private List<String> termStep2Notes = new ArrayList<>();

	// Step1 回跳兜底
	private String step1Url;
	private String step1Handle;

	// NEW: 動態 rowId（例如 r80241059 -> 80241059）
	private String detectedRowId; // 來源以 Step2 的第一列 r{rowId} 偵測

	public CSROneCallPass() {
		pageDisplayCorrectlyLocator = By
				.xpath("//th[text()='選擇異動保單']/ancestor::table[@class='table table-bordered search-table']");
	}

	@Override
	public boolean isPageDisplayCorrectly(int waitingTime) {
		try {
			waitForWindowHandlesToSettle(600, 8000);
			switchToMostRecentAliveWindow();

			// Step1
			WebElement displayResult = findElement(pageDisplayCorrectlyLocator, false, waitingTime);
			if (displayResult == null) return false;
			WebElement displayResult2 = findElement(selectChangeItemOptLocator, false, waitingTime);
			if (displayResult2 == null) return false;

			captureStep1Context(); // NEW: 抓主分頁資訊（URL/handle）

			List<String> expectedTexts = Arrays.asList("選項","保單號碼","契約始期下次繳費日","要保人被保險人","主約險種","幣別","繳別","每期保費","收費管道保單狀態","繳費年期停效日","服務銀行/單位");
			new CommonPage().verifyTableTitle(expectedTexts, selectPolicyChangeTableMenuLocator);

			// 先勾保單
			if (!ensureChecked(By.id("checkbox" + POLICY_NO), By.xpath(policyChangeCheckboxLocator), 6000)) return false;

			// ====== 【保單借款】流程 ======
			WebElement policyLoanRadioButton = findElementIsDisplayed(By.xpath(policyLoanRadioButtonLocator), true, 5000);
			if (policyLoanRadioButton == null) return false;
			click(policyLoanRadioButton);

			WebElement stepOneNextButton = findElementIsDisplayed(By.xpath(stepOneNextButtonLocator), true, 5000);
			if (stepOneNextButton == null) return false;
			click(stepOneNextButton);

			waitForStable(); // NEW: 等待 Step2 載入穩定

			// Step2（借款）
			expectedTexts = Arrays.asList("保單號碼","本月借款利率","最高可借款金額","已借款本息","可借款餘額","借款類型","本次借款金額","借款原因");
			if (!verifyTableTitleLoose(expectedTexts, enterLoanContentTableTitleLocator)) return false;

			// NEW: 偵測 rowId（如 80241059），後續欄位用動態 id
			detectedRowId = detectRowId();
			if (detectedRowId == null || detectedRowId.isBlank()) detectedRowId = POLICY_NO;

			try {
				List<WebElement> policyChangeTableDatas = findElements(By.xpath(policyChangeTableDataXpath()), 5000);
				stepTwoPolicyRow.clear();
				for (WebElement cell : policyChangeTableDatas) stepTwoPolicyRow.add(safeText(cell));
			} catch (Exception e) { return false; }

			try {
				List<WebElement> loanRowCells = findElements(By.xpath(enterLoanContentTableDataXpath()), 5000);
				stepTwoLoanRowHead5.clear();
				for (int i = 0; i < Math.min(5, loanRowCells.size()); i++) stepTwoLoanRowHead5.add(safeText(loanRowCells.get(i)));
			} catch (Exception e) { return false; }

			// 借款類型
			{
				final String TARGET_VALUE = "1";
				final String TARGET_TEXT  = "一般借款";
				String selectedValue = selectFromSelectOrBootstrap(loanTypeSelectBy(), TARGET_VALUE, TARGET_TEXT);
				if (selectedValue == null) {
					selectedValue = selectFromSelectOrBootstrap(By.xpath(loanTypeDropDownlistLocator), TARGET_VALUE, TARGET_TEXT);
				}
				if (selectedValue == null) return false;
				selectedLoanTypeValue = selectedValue.trim();
				selectedLoanTypeText  = getSelectedText(loanTypeSelectBy());
			}
			// 借款原因
			{
				final String TARGET_VALUE = "A";
				final String TARGET_TEXT  = "投資理財";
				String selectedValue = selectFromSelectOrBootstrap(loanReasonSelectBy(), TARGET_VALUE, TARGET_TEXT);
				if (selectedValue == null) {
					selectedValue = selectFromSelectOrBootstrap(By.xpath(loanReasonDropdownLocator), TARGET_VALUE, TARGET_TEXT);
				}
				if (selectedValue == null) return false;
				selectedLoanReasonValue = selectedValue.trim();
				selectedLoanReasonText  = getSelectedText(loanReasonSelectBy());
			}
			// 金額
			WebElement currentLoanAmount = findElementIsDisplayed(currentLoanAmountInputBy(), true, 5000);
			if (currentLoanAmount == null) return false;
			input(currentLoanAmount, "40000");
			inputLoanAmount = "40000";

			// 付款方式
			WebElement radio = findElementIsDisplayed(useProvidedRadio, true, 5000);
			if (radio == null) return false;
			click(radio, true);
			{
				final String TARGET_VALUE = "台灣工銀營業部/01000020231800/0480011";
				final String TARGET_TEXT  = "台灣工銀營業部  01000020231800  滿Ｑ";
				String selectedValue = selectFromSelectOrBootstrap(providedAccountSelect, TARGET_VALUE, TARGET_TEXT);
				if (selectedValue == null) {
					selectedValue = selectFirstOptionFromSelectOrBootstrap(providedAccountSelect);
					if (selectedValue == null) return false;
				}
			}
			selectedProvidedAcctText = getSelectedText(providedAccountSelect);

			// 收件人資訊
			WebElement chargeAddrRadio = findElementIsDisplayed(chargeAddressRadio, true, 5000);
			if (chargeAddrRadio == null) return false;
			click(chargeAddrRadio, true);
			WebElement chargeAddrInput = findElementIsDisplayed(chargeAddressInput, false, 5000);
			if (chargeAddrInput == null) return false;
			chargeAddressFull = chargeAddrInput.getAttribute("value");
			recipientNameText = getSelectedText(recipientNameSelect);
			mailTypeText      = getSelectedText(mailTypeSelect);

			// 文件/提醒 → 若有就全勾
			checkAllIfExists(By.cssSelector("#tb_doc tbody input[type='checkbox']"));
			checkAllIfExists(By.cssSelector("#tb_note tbody input[type='checkbox']"));

			// 收集勾選結果
			stepTwoDocs  = collectCheckedTextsFromTable("tb_doc", 3);
			stepTwoNotes = collectCheckedTextsFromTable("tb_note", 3);

			waitForStable();

			// 進 Step3（借款）
			boolean loanOk = goToStepThreeAndVerifyLoan();
			// 無論驗證結果如何，先嘗試回 Step1，避免場景卡死
			if (!returnToStep1Robust("loan-review")) return false;
			if (!loanOk) return false;

			// ===== 契約終止流程 =====
			// 再次勾保單（預防返回後狀態被重置）
			if (!ensureChecked(By.id("checkbox" + POLICY_NO), By.xpath(policyChangeCheckboxLocator), 6000)) return false;

			WebElement terminiateContractRadioButton = findElementIsDisplayed(By.xpath(terminiateContractRadioButtonLocator), true, 5000);
			if (terminiateContractRadioButton == null) return false;
			click(terminiateContractRadioButton);

			WebElement stepOneNextButtonSecTime = findElementIsDisplayed(By.xpath(stepOneNextButtonLocator), true, 5000);
			if (stepOneNextButtonSecTime == null) return false;
			click(stepOneNextButtonSecTime);

			waitForStable();

			expectedTexts = Arrays.asList("保單號碼","契約始期下次繳費日","要保人被保險人","主約險種","幣別","繳別","每期保費","收費管道保單狀態","繳費年期停效日","服務銀行/單位");
			if (!verifyTableTitleLoose(expectedTexts, terminateContractStep2ChangePolicyTableHeadersLocator)) return false;

			// 重新偵測 rowId
			String termRowId = detectRowId();
			if (termRowId == null || termRowId.isBlank()) termRowId = getRowId();
			setRowId(termRowId);

			try {
				List<WebElement> policyChangeTableData = findElements(By.xpath(terminateContractStep2ChangePolicyTableDataXpath()), 5000);
				termStep2PolicyRow.clear();
				for (WebElement cell : policyChangeTableData) termStep2PolicyRow.add(safeText(cell));
			} catch (Exception e) { return false; }

			expectedTexts = Arrays.asList("保單號碼","解約原因");
			if (!verifyTableTitleLoose(expectedTexts, cancellationReasonTableHeadersLocator)) return false;

			// 解約原因
			By reasonSelectLoose = By.xpath("(//table[@id='terminate2']//select)[1]");
			String reasonSelected = selectByTextFromSelectOrBootstrap(reasonSelectLoose, "資金調度需求");
			if (reasonSelected == null || reasonSelected.isBlank()) {
				reasonSelected = selectFirstOptionFromSelectOrBootstrap(reasonSelectLoose);
			}
			if (reasonSelected == null) return false;
			termSelectedReasonText = getSelectedText(reasonSelectLoose);

			// 付款方式
			WebElement tRadio = findElementIsDisplayed(useProvidedRadio, true, 5000);
			if (tRadio == null) return false;
			click(tRadio, true);
			String val = selectFirstOptionFromSelectOrBootstrap(providedAccountSelect);
			if (val == null) return false;
			termSelectedProvidedAcctText = getSelectedText(providedAccountSelect);

			// 收件人資訊
			WebElement tAddrRadio = findElementIsDisplayed(chargeAddressRadio, true, 5000);
			if (tAddrRadio == null) return false;
			click(tAddrRadio, true);
			WebElement tAddrInput = findElementIsDisplayed(chargeAddressInput, false, 5000);
			if (tAddrInput == null) return false;
			termChargeAddressFull = tAddrInput.getAttribute("value");
			termRecipientNameText = getSelectedText(recipientNameSelect);
			termMailTypeText      = getSelectedText(mailTypeSelect);

			// 文件/提醒 → 也全勾
			checkAllIfExists(By.cssSelector("#tb_doc tbody input[type='checkbox']"));
			checkAllIfExists(By.cssSelector("#tb_note tbody input[type='checkbox']"));

			termStep2Docs  = collectCheckedTextsFromTable("tb_doc", 3);
			termStep2Notes = collectCheckedTextsFromTable("tb_note", 3);

			waitForStable();

			// 進 Step3（契約終止）
			boolean termOk = goToStepThreeAndVerifyTerminate();

			// ★ 不論驗證是否成功，都嘗試回 Step1（關鍵修正）
			boolean backOk = returnToStep1Robust("terminate-review");

			// 若驗證或回跳有任何一個失敗，都視為整體失敗
			if (!termOk || !backOk) return false;

			return true;
		} finally {
			switchToSurvivorWindow();
		}
	}

	// ====== Step3（保單借款） ======
	private boolean goToStepThreeAndVerifyLoan() {
		WebElement reviewBtn = findElementIsDisplayed(reviewButtonXPath, true, 6000);
		if (reviewBtn == null) reviewBtn = findElementIsDisplayed(reviewButtonId, true, 4000);
		if (reviewBtn == null) return false;

		// UPDATED: 點「預覽」→ 允許新分頁→等它自動關→回主分頁
		if (!clickAndHandleShortLivedWindow(reviewBtn, 2000, 12000)) return false;

		// 回主分頁後定位 Step3
		waitForWindowHandlesToSettle(700, 12000);
		refocusMainWindow(); // NEW
		if (!locateStep3Root("loan")) return false;

		boolean ok = true;

		try {
			List<WebElement> s3LoanCells = findElements(By.xpath("//table[@id='loanData3']//tr[contains(@class,'pls')]/td"), 5000);
			if (s3LoanCells.size() < 8) return false;
			String s3PolicyNo   = safeText(s3LoanCells.get(0));
			String s3Rate       = safeText(s3LoanCells.get(1));
			String s3MaxLoan    = safeText(s3LoanCells.get(2));
			String s3Borrowed   = safeText(s3LoanCells.get(3));
			String s3Balance    = safeText(s3LoanCells.get(4));
			String s3LoanTypeTx = safeText(s3LoanCells.get(5));
			String s3Amount     = safeText(s3LoanCells.get(6));
			String s3ReasonTx   = safeText(s3LoanCells.get(7));

			ok &= eqText(stepTwoLoanRowHead5.get(0), s3PolicyNo,   "保單號碼");
			ok &= eqNumber(stepTwoLoanRowHead5.get(1), s3Rate,     "本月借款利率");
			ok &= eqNumber(stepTwoLoanRowHead5.get(2), s3MaxLoan,  "最高可借款金額");
			ok &= eqNumber(stepTwoLoanRowHead5.get(3), s3Borrowed, "已借款本息");
			ok &= eqNumber(stepTwoLoanRowHead5.get(4), s3Balance,  "可借款餘額");

			ok &= eqText(selectedLoanTypeText,  s3LoanTypeTx, "借款類型(顯示)");
			ok &= eqNumber(inputLoanAmount,     s3Amount,     "本次借款金額");
			ok &= eqText(selectedLoanReasonText, s3ReasonTx,  "借款原因(顯示)");
		} catch (Exception e) { return false; }

		try {
			WebElement tdTtransfer = findElementIsDisplayed(By.id("tdTtransfer"), true, 5000);
			if (tdTtransfer == null) return false;
			ok &= containsText(safeText(tdTtransfer), selectedProvidedAcctText, "付款方式-帳號顯示");
		} catch (Exception e) { return false; }

		try {
			String s3Name = safeText(findElementIsDisplayed(By.id("tdclinetName"), true, 5000));
			String s3Mail = safeText(findElementIsDisplayed(By.id("tdmailType"),  true, 5000));
			String s3Addr = safeText(findElementIsDisplayed(By.id("tdsendForm"),  true, 5000));
			ok &= eqText(recipientNameText, s3Name, "收件人姓名");
			ok &= eqText(mailTypeText,      s3Mail, "郵件別");
			ok &= eqText(chargeAddressFull, s3Addr, "寄送地址");
		} catch (Exception e) { return false; }

		try {
			WebElement docCell  = findElement(By.id("tddoc"),  false, 3000);
			WebElement noteCell = findElement(By.id("tdnote"), false, 3000);
			String s3Docs  = docCell  != null ? safeText(docCell)  : "";
			String s3Notes = noteCell != null ? safeText(noteCell) : "";
			for (String d : stepTwoDocs)  ok &= containsText(s3Docs,  d, "應檢附文件");
			for (String n : stepTwoNotes) ok &= containsText(s3Notes, n, "提醒事項");
		} catch (Exception e) { return false; }

		return ok;
	}

	// ====== Step3（契約終止） ======
	private boolean goToStepThreeAndVerifyTerminate() {
		WebElement reviewBtn = findElementIsDisplayed(reviewButtonXPath, true, 6000);
		if (reviewBtn == null) reviewBtn = findElementIsDisplayed(reviewButtonId, true, 4000);
		if (reviewBtn == null) return false;

		// UPDATED: 同樣用短命視窗處理
		if (!clickAndHandleShortLivedWindow(reviewBtn, 2000, 12000)) return false;

		waitForWindowHandlesToSettle(700, 12000);
		refocusMainWindow(); // NEW
		if (!locateStep3Root("terminate")) return false;

		boolean ok = true;
		String pageText = getWholePageText();

		// 放寬比對：僅核對保單號碼（避免因欄位格式/空白差異造成早退）
		try {
			String policyNoFromStep2 = termStep2PolicyRow.isEmpty() ? "" : normalizeText(termStep2PolicyRow.get(0));
			if (!policyNoFromStep2.isEmpty()) {
				ok &= containsText(pageText, policyNoFromStep2, "契約終止-保單號碼");
			}
		} catch (Exception ignore) {}

		// 解約原因
		ok &= containsText(pageText, termSelectedReasonText, "解約原因(顯示)");

		// 付款方式
		try {
			WebElement tdTtransfer = findElementIsDisplayed(By.id("tdTtransfer"), true, 3000);
			String s3PayText = tdTtransfer != null ? safeText(tdTtransfer) : pageText;
			ok &= containsText(s3PayText, termSelectedProvidedAcctText, "付款方式-帳號顯示(契約終止)");
		} catch (Exception ignore) {}

		// 收件人資訊
		try {
			String s3Name = tryGetText(By.id("tdclinetName"));
			String s3Mail = tryGetText(By.id("tdmailType"));
			String s3Addr = tryGetText(By.id("tdsendForm"));
			if (!s3Name.isEmpty()) ok &= eqText(termRecipientNameText, s3Name, "收件人姓名(契約終止)");
			if (!s3Mail.isEmpty()) ok &= eqText(termMailTypeText,      s3Mail, "郵件別(契約終止)");
			if (!s3Addr.isEmpty()) ok &= eqText(termChargeAddressFull, s3Addr, "寄送地址(契約終止)");
		} catch (Exception ignore) {}

		// 檢附文件 / 提醒事項
		try {
			WebElement docCell  = findElement(By.id("tddoc"),  false, 3000);
			WebElement noteCell = findElement(By.id("tdnote"), false, 3000);
			String s3Docs  = docCell  != null ? safeText(docCell)  : pageText;
			String s3Notes = noteCell != null ? safeText(noteCell) : pageText;
			for (String d : termStep2Docs)  ok &= containsText(s3Docs,  d, "契約終止-應檢附文件");
			for (String n : termStep2Notes) ok &= containsText(s3Notes, n, "契約終止-提醒事項");
		} catch (Exception ignore) {}

		return ok;
	}

	// ====== 回 Step1：加「滾動脈衝 + 多策略返回 + 反抖動視窗切換」 ======
	private boolean returnToStep1Robust(String phaseTag) {
		waitForWindowHandlesToSettle(800, 10000);
		switchToMostRecentAliveWindow();
		try { Thread.sleep(400); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }

		for (int round = 1; round <= 3; round++) {
			logger.debug("[returnToStep1Robust] round=" + round + " phase=" + phaseTag);

			// NEW: 先對目前 window / 其頂層 iframe 做「滾動脈衝」，觸發 lazy render
			try { pulseScrollGloballyAndTopIframes(); } catch (Exception ignore) {}

			// 1) 直接點返回控制（目前 context）
			switchToMostRecentAliveWindow();
			if (tryClickAnyReturnControlInCurrentContext()) {
				waitForWindowHandlesToSettle(800, 10000);
				switchToMostRecentAliveWindow();
				if (waitStep1Visible(15000)) return true;
			}

			// 2) 到每個 top-level iframe 嘗試（每個 frame 先滾動再找）
			switchToMostRecentAliveWindow();
			if (tryClickReturnControlsInAllTopFramesWithScroll()) {
				waitForWindowHandlesToSettle(800, 10000);
				switchToMostRecentAliveWindow();
				if (waitStep1Visible(15000)) return true;
			}

			// 3) JS back（有些 SPA 攔截原生 back）
			switchToMostRecentAliveWindow();
			try {
				((JavascriptExecutor)driver).executeScript("try{history.go(-1);}catch(e){}");
			} catch (Exception ignore) {}
			waitForWindowHandlesToSettle(600, 8000);
			switchToMostRecentAliveWindow();
			if (waitStep1Visible(6000)) return true;

			// 4) 瀏覽器 back（最多 2 次）
			if (browserBackUntilStep1(2, 5000)) return true;

			// 5) 兜底：導回 Step1 URL（每次都先切到活視窗再試）
			if (step1Url != null && !step1Url.isBlank()) {
				switchToMostRecentAliveWindow();
				try {
					driver.navigate().to(step1Url);
					waitForWindowHandlesToSettle(800, 10000);
					switchToMostRecentAliveWindow();
					if (waitStep1Visible(15000)) return true;
				} catch (Exception e) {
					logger.debug("[returnToStep1Robust] navigate().to(step1Url) 失敗: " + e);
				}
			}

			try { Thread.sleep(500); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
		}
		logger.debug("[returnToStep1Robust] 無法回到 Step1（phase=" + phaseTag + "）");
		return false;
	}

	private boolean waitStep1Visible(int timeoutMs) {
		long end = System.currentTimeMillis() + timeoutMs;
		while (System.currentTimeMillis() < end) {
			try {
				if (findElement(selectChangeItemOptLocator, false, 800) != null &&
					findElement(pageDisplayCorrectlyLocator,  false, 800) != null) {
					return true;
				}
			} catch (Exception ignore) {}
			try { Thread.sleep(200); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
		}
		return false;
	}

	/** 等分頁數穩定（開/關的抖動都涵蓋） */
	private boolean waitForWindowHandlesToSettle(int settleMillis, int timeoutMillis) {
		long end = System.currentTimeMillis() + timeoutMillis;
		int lastCount = -1;
		long lastChangeTs = System.currentTimeMillis();
		while (System.currentTimeMillis() < end) {
			try {
				Set<String> handles = driver.getWindowHandles();
				int nowCount = handles.size();
				if (nowCount != lastCount) {
					lastCount = nowCount;
					lastChangeTs = System.currentTimeMillis();
				} else {
					if (System.currentTimeMillis() - lastChangeTs >= settleMillis) return true;
				}
				Thread.sleep(180);
			} catch (Exception ignore) {
				try { Thread.sleep(220); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
			}
		}
		return false;
	}

	private boolean switchToMostRecentAliveWindow() {
		try {
			List<String> order = new ArrayList<>(driver.getWindowHandles());
			for (int i = order.size() - 1; i >= 0; i--) {
				String h = order.get(i);
				try {
					driver.switchTo().window(h);
					driver.getTitle(); // 觸發連線；死掉會丟例外
					driver.switchTo().defaultContent();
					return true;
				} catch (Exception ignore) {}
			}
		} catch (Exception ignore) {}
		return false;
	}

	// ====== 勾選 checkbox ======
	private boolean ensureChecked(By inputBy, By labelBy, int timeoutMs) {
		long end = System.currentTimeMillis() + timeoutMs;
		JavascriptExecutor js = (JavascriptExecutor) driver;
		WebElement label = findElementIsDisplayed(labelBy, true, Math.min(4000, timeoutMs));
		WebElement input = findElement(inputBy, false, 1000);
		if (input == null && label != null) {
			try {
				String fid = label.getAttribute("for");
				if (fid != null && !fid.isEmpty()) input = findElement(By.id(fid), false, 800);
			} catch (Exception ignore) {}
		}
		if (isCheckboxSelected(input, label)) return true;

		boolean clicked = false;
		if (label != null) {
			try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", label); } catch (Exception ignore) {}
			try { click(label, true); clicked = true; } catch (Exception e) { try { js.executeScript("arguments[0].click();", label); clicked = true; } catch (Exception ignore) {} }
		}
		if (!isCheckboxSelected(input, label) && input != null) {
			try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", input); } catch (Exception ignore) {}
			try { click(input, true); clicked = true; } catch (Exception e) { try { js.executeScript("arguments[0].click();", input); clicked = true; } catch (Exception ignore) {} }
		}
		try { Thread.sleep(400); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }

		if (input == null && label != null) {
			try {
				String fid = label.getAttribute("for");
				if (fid != null && !fid.isEmpty()) input = findElement(By.id(fid), false, 500);
			} catch (Exception ignore) {}
		}
		if (isCheckboxSelected(input, label)) return true;
		if (clicked) return true;

		WebElement row = findRowByPolicyNoLoose();
		if (row != null) {
			try {
				WebElement any = null;
				try { any = row.findElement(By.xpath(".//label[contains(@for,'checkbox') or self::label]")); } catch (Exception ignore) {}
				if (any == null) { try { any = row.findElement(By.xpath(".//input[@type='checkbox']")); } catch (Exception ignore) {} }
				if (any != null) {
					try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", any); } catch (Exception ignore) {}
					try { any.click(); } catch (Exception e) { try { js.executeScript("arguments[0].click();", any); } catch (Exception ignore) {} }
					return true;
				}
			} catch (Exception ignore) {}
		}
		return false;
	}

	private boolean isCheckboxSelected(WebElement input, WebElement label) {
		try { if (input != null) return input.isSelected(); } catch (Exception ignore) {}
		try {
			if (label != null) {
				String fid = label.getAttribute("for");
				if (fid != null && !fid.isEmpty()) {
					WebElement again = findElement(By.id(fid), false, 500);
					if (again != null) return again.isSelected();
				}
			}
		} catch (Exception ignore) {}
		try {
			WebElement scope = null;
			if (label != null) scope = label.findElement(By.xpath("ancestor::tr[1]"));
			if (scope != null) {
				WebElement cb = null;
				try { cb = scope.findElement(By.xpath(".//input[@type='checkbox']")); } catch (Exception ignore) {}
				if (cb != null) return cb.isSelected();
			}
		} catch (Exception ignore) {}
		try {
			if (label != null) {
				String cls = label.getAttribute("class");
				if (cls != null && (cls.contains("active") || cls.contains("checked"))) return true;
				String aria = label.getAttribute("aria-checked");
				if ("true".equalsIgnoreCase(aria)) return true;
			}
		} catch (Exception ignore) {}
		return false;
	}

	// ====== Bootstrap / 原生 select：強化（支援無 id 的 select） ======

	private String selectFromSelectOrBootstrap(By selectBy, String targetValue, String targetText) {
		if (targetValue == null || targetValue.isBlank()) {
			return selectByTextFromSelectOrBootstrap(selectBy, targetText);
		}
		WebElement selectEl = findElementIsDisplayed(selectBy, true, 5000);
		if (selectEl == null) return null;

		WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
		JavascriptExecutor js = (JavascriptExecutor) driver;

		boolean usedBootstrap = false;
		WebElement toggle = findBootstrapToggle(selectEl);
		if (toggle != null) {
			usedBootstrap = true;
			openDropdown(toggle, wait, js);
			WebElement bsContainer = toggle.findElement(By.xpath("ancestor::div[contains(@class,'bootstrap-select')]"));
			WebElement menu = bsContainer.findElement(By.cssSelector(".dropdown-menu"));
			wait.until(ExpectedConditions.visibilityOf(menu));
			try {
				WebElement option = menu.findElement(By.xpath(".//span[@class='text' and normalize-space()='" + targetText + "']/ancestor::a"));
				try { option.click(); } catch (Exception e) { js.executeScript("arguments[0].click();", option); }
			} catch (Exception e) {
				try {
					WebElement any = menu.findElement(By.xpath(".//a"));
					js.executeScript("arguments[0].click();", any);
				} catch (Exception ignore) {}
			}
		}
		if (!usedBootstrap) {
			try { new Select(selectEl).selectByValue(targetValue); }
			catch (Exception e) {
				js.executeScript("arguments[0].value = arguments[1]; arguments[0].dispatchEvent(new Event('change', {bubbles:true}));",
						selectEl, targetValue);
			}
		}
		String finalVal = selectEl.getAttribute("value");
		if (finalVal == null || finalVal.isBlank()) {
			finalVal = getSelectedText(selectBy); // 至少回傳顯示文字
		}
		return finalVal;
	}

	public String selectByTextFromSelectOrBootstrap(By selectBy, String targetText) {
		WebElement selectEl = findElementIsDisplayed(selectBy, true, 5000);
		if (selectEl == null) return null;

		WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
		JavascriptExecutor js = (JavascriptExecutor) driver;

		boolean usedBootstrap = false;
		WebElement toggle = findBootstrapToggle(selectEl); // 就近找 toggle（即便 select 沒 id）
		if (toggle != null) {
			usedBootstrap = true;
			openDropdown(toggle, wait, js);
			try {
				WebElement bsContainer = toggle.findElement(By.xpath("ancestor::div[contains(@class,'bootstrap-select')]"));
				WebElement menu = bsContainer.findElement(By.cssSelector(".dropdown-menu"));
				wait.until(ExpectedConditions.visibilityOf(menu));
				// 先精準，再寬鬆 contains
				List<By> tryLocators = Arrays.asList(
					By.xpath(".//span[@class='text' and normalize-space()='" + targetText + "']/ancestor::a"),
					By.xpath(".//span[@class='text'][contains(normalize-space(),'" + targetText + "')]/ancestor::a")
				);
				boolean clicked = false;
				for (By by : tryLocators) {
					try {
						WebElement opt = menu.findElement(by);
						try { opt.click(); } catch (Exception e) { js.executeScript("arguments[0].click();", opt); }
						clicked = true; break;
					} catch (Exception ignore) {}
				}
				if (!clicked) {
					// 後援：選第一個有效
					List<WebElement> items = menu.findElements(By.xpath(".//a[.//span[@class='text' and normalize-space()!='' and not(contains(.,'請選'))]]"));
					if (!items.isEmpty()) js.executeScript("arguments[0].click();", items.get(0));
				}
			} catch (Exception ignore) {}
		}
		if (!usedBootstrap) {
			try { new Select(selectEl).selectByVisibleText(targetText); }
			catch (Exception e) {
				// 後援：contains 或第一個有效
				try {
					Select s = new Select(selectEl);
					boolean found = false;
					for (WebElement opt : s.getOptions()) {
						String t = safeText(opt);
						if (t.contains(targetText) && t.length()>0) { s.selectByVisibleText(t); found = true; break; }
					}
					if (!found) {
						for (WebElement opt : s.getOptions()) {
							String t = safeText(opt);
							if (!t.isBlank() && !t.contains("請選")) { s.selectByVisibleText(t); break; }
						}
					}
				} catch (Exception ignore) { return null; }
			}
		}
		return getSelectedText(selectBy);
	}

	private WebElement findBootstrapToggle(WebElement selectEl) {
		// 1) 以 id 對應 data-id
		try {
			String id = selectEl.getAttribute("id");
			if (id != null && !id.isBlank()) {
				List<WebElement> toggles = driver.findElements(By.cssSelector("button.dropdown-toggle[data-id='" + id + "']"));
				for (WebElement t : toggles) if (t.isDisplayed()) return t;
			}
		} catch (Exception ignore) {}
		// 2) 無 id：找緊鄰的 bootstrap-select 容器
		try {
			WebElement t = selectEl.findElement(By.xpath("following-sibling::div[contains(@class,'bootstrap-select')]//button[contains(@class,'dropdown-toggle')]"));
			if (t != null && t.isDisplayed()) return t;
		} catch (Exception ignore) {}
		// 3) 向上找就近容器再往下找
		try {
			List<WebElement> cand = selectEl.findElements(By.xpath("ancestor::*[self::td or self::div or self::span][1]//button[contains(@class,'dropdown-toggle')]"));
			for (WebElement t : cand) if (t.isDisplayed()) return t;
		} catch (Exception ignore) {}
		return null;
	}

	private void openDropdown(WebElement toggle, WebDriverWait wait, JavascriptExecutor js) {
		try { wait.until(ExpectedConditions.elementToBeClickable(toggle)).click(); }
		catch (Exception e) { js.executeScript("arguments[0].click();", toggle); }
	}

	// 選第一個有效選項
	private String selectFirstOptionFromSelectOrBootstrap(By selectBy) {
		WebElement sel = findElementIsDisplayed(selectBy, true, 5000);
		if (sel == null) return null;
		try {
			Select s = new Select(sel);
			for (WebElement opt : s.getOptions()) {
				String t = opt.getText() == null ? "" : opt.getText().trim();
				String v = opt.getAttribute("value");
				if (t.isBlank() || v == null || v.isBlank() || t.contains("請選") || t.contains("Select")) continue;
				s.selectByValue(v);
				return v;
			}
		} catch (Exception e) {
			try {
				WebElement toggle = findBootstrapToggle(sel);
				if (toggle != null) {
					openDropdown(toggle, new WebDriverWait(driver, Duration.ofSeconds(8)), (JavascriptExecutor)driver);
					WebElement menu = toggle.findElement(By.xpath("ancestor::div[contains(@class,'bootstrap-select')]")).findElement(By.cssSelector(".dropdown-menu"));
					List<WebElement> items = menu.findElements(By.xpath(".//a[.//span[@class='text' and normalize-space()!='' and not(contains(.,'請選'))]]"));
					if (!items.isEmpty()) {
						((JavascriptExecutor)driver).executeScript("arguments[0].click();", items.get(0));
						return getSelectedText(selectBy);
					}
				}
			} catch (Exception ignore) {}
		}
		return getSelectedText(selectBy);
	}

	// ====== 小工具 ======
	private void captureStep1Context() {
		try { step1Url = driver.getCurrentUrl(); } catch (Exception ignore) {}
		try { step1Handle = driver.getWindowHandle(); } catch (Exception ignore) {}
	}

	/** 優先嘗試：點擊第 2 個「取消申請」按鈕（若存在） */
	private boolean clickSecondCancelIfPresent() {
		JavascriptExecutor js = (JavascriptExecutor) driver;
		// 嚴格匹配：class 完全等於 "btn btn-default"
		By strictSecond = By.xpath("(//a[@class='btn btn-default' and normalize-space(text())='取消申請'])[2]");
		try {
			WebElement btn = findElementIsDisplayed(strictSecond, true, 1000);
			if (btn != null) {
				try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", btn); } catch (Exception ignore) {}
				try { click(btn, true); } catch (Exception e) { js.executeScript("arguments[0].click();", btn); }
				return true;
			}
		} catch (Exception ignore) {}

		// 寬鬆匹配：class token 包含 btn 與 btn-default
		By looseSecond = By.xpath("(//a[contains(concat(' ',normalize-space(@class),' '),' btn ') and contains(concat(' ',normalize-space(@class),' '),' btn-default ') and normalize-space(text())='取消申請'])[2]");
		try {
			WebElement btn = findElementIsDisplayed(looseSecond, true, 800);
			if (btn != null) {
				try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", btn); } catch (Exception ignore) {}
				try { click(btn, true); } catch (Exception e) { js.executeScript("arguments[0].click();", btn); }
				return true;
			}
		} catch (Exception ignore) {}

		// 最後備援：收集全部「取消申請」，若 >=2 就點第 2 個
		try {
			List<WebElement> all = findElements(By.xpath("//a[@class='btn btn-default' and normalize-space(text())='取消申請']"), 800);
			if (all != null && all.size() >= 2) {
				WebElement btn = all.get(1);
				try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", btn); } catch (Exception ignore) {}
				try { click(btn, true); } catch (Exception e) { js.executeScript("arguments[0].click();", btn); }
				return true;
			}
		} catch (Exception ignore) {}

		// 亦支援 <button> 節點
		try {
			WebElement btn = findElementIsDisplayed(By.xpath("(//button[contains(@class,'btn') and contains(@class,'btn-default') and normalize-space(text())='取消申請'])[2]"), true, 800);
			if (btn != null) {
				try { js.executeScript("arguments[0].scrollIntoView({block:'center'});", btn); } catch (Exception ignore) {}
				try { click(btn, true); } catch (Exception e) { js.executeScript("arguments[0].click();", btn); }
				return true;
			}
		} catch (Exception ignore) {}

		return false;
	}

	private boolean tryClickAnyReturnControlInCurrentContext() {
		// 先嘗試：第 2 個「取消申請」
		if (clickSecondCancelIfPresent()) return true;

		By[] backBtnLocators = new By[] {
			// 指定第 2 個取消鈕（備援；前面的專用方法已先嘗試過）
			By.xpath("(//a[@class='btn btn-default' and normalize-space()='取消申請'])[2]"),
			// 其餘常見返回鍵
			By.xpath("//a[@class='btn btn-default' and normalize-space()='取消申請']"),
			By.xpath("//a[contains(@class,'btn') and contains(normalize-space(),'取消申請')]"),
			By.xpath("(//button[contains(@class,'btn') and contains(@class,'btn-default') and normalize-space()='取消申請'])[1]"),
			By.id("cancel"),
			By.xpath("//a[contains(@class,'btn') and (contains(normalize-space(),'回上一步') or contains(normalize-space(),'上一步') or contains(normalize-space(),'返回') or contains(normalize-space(),'返回修正') or contains(normalize-space(),'回列表') or contains(normalize-space(),'關閉'))]"),
			By.xpath("//button[contains(@class,'btn') and (contains(normalize-space(),'回上一步') or contains(normalize-space(),'上一步') or contains(normalize-space(),'返回') or contains(normalize-space(),'返回修正') or contains(normalize-space(),'回列表') or contains(normalize-space(),'關閉'))]"),
			By.xpath("//ul[contains(@class,'steps') or contains(@class,'wizard') or contains(@class,'process')]//a[contains(.,'1') or contains(.,'選擇異動保單')]")
		};
		for (By by : backBtnLocators) {
			try {
				WebElement btn = findElementIsDisplayed(by, true, 2500);
				if (btn != null) {
					try { ((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView({block:'center'});", btn); } catch (Exception ignore) {}
					try { click(btn, true); } catch (Exception e) { ((JavascriptExecutor) driver).executeScript("arguments[0].click();", btn); }
					return true;
				}
			} catch (Exception ignore) {}
		}
		return false;
	}

	private boolean tryClickReturnControlsInAllTopFramesWithScroll() {
		try {
			driver.switchTo().defaultContent();
			List<WebElement> ifs = driver.findElements(By.tagName("iframe"));
			for (int i = 0; i < ifs.size(); i++) {
				try {
					driver.switchTo().frame(i);
					// 每個 frame 先做滾動脈衝，再找返回鍵
					try { pulseScrollCurrentContext(); } catch (Exception ignore) {}
					if (tryClickAnyReturnControlInCurrentContext()) {
						driver.switchTo().defaultContent();
						return true;
					}
					driver.switchTo().defaultContent();
				} catch (Exception e) {
					driver.switchTo().defaultContent();
				}
			}
		} catch (Exception ignore) {}
		return false;
	}

	private boolean browserBackUntilStep1(int maxBack, int waitEachMs) {
		for (int i = 1; i <= maxBack; i++) {
			try {
				switchToMostRecentAliveWindow();
				driver.navigate().back();
				waitForWindowHandlesToSettle(600, 8000);
				switchToMostRecentAliveWindow();
				if (waitStep1Visible(Math.max(2500, waitEachMs))) return true;
				Thread.sleep(180);
			} catch (Exception ignore) {}
		}
		return false;
	}

	private void switchToSurvivorWindow() {
		try { driver.getTitle(); driver.switchTo().defaultContent(); return; }
		catch (Exception ignore) {}
		try {
			for (String h : driver.getWindowHandles()) {
				try {
					driver.switchTo().window(h);
					driver.getTitle();
					driver.switchTo().defaultContent();
					return;
				} catch (Exception ignore) {}
			}
		} catch (Exception ignore) {}
		try {
			driver.switchTo().newWindow(WindowType.TAB);
			driver.navigate().to("about:blank");
			driver.switchTo().defaultContent();
		} catch (Exception ignore) {}
	}

	private String safeText(WebElement el) {
		if (el == null) return "";
		return el.getText() == null ? "" : el.getText().trim();
	}

	private String getSelectedText(By selectBy) {
		WebElement sel = findElementIsDisplayed(selectBy, true, 3000);
		if (sel == null) return "";
		try { return new Select(sel).getFirstSelectedOption().getText().trim(); }
		catch (Exception e) {
			try {
				String id = sel.getAttribute("id");
				if (id != null && !id.isBlank()) {
					WebElement toggleText = driver.findElement(
						By.cssSelector("button.dropdown-toggle[data-id='" + id + "'] .filter-option-inner-inner"));
					return toggleText.getText().trim();
				}
			} catch (Exception ignore) {}
			// 無 id 的 bootstrap：就近找
			try {
				WebElement toggle = findBootstrapToggle(sel);
				if (toggle != null) {
					WebElement t = toggle.findElement(By.cssSelector(".filter-option-inner-inner"));
					return safeText(t);
				}
			} catch (Exception ignore) {}
			return "";
		}
	}

	private boolean eqText(String s2, String s3, String label) {
		String a = normalizeText(s2);
		String b = normalizeText(s3);
		boolean same = a.equals(b);
		if (!same) logger.debug("[不一致] " + label + " | S2='" + s2 + "' vs S3='" + s3 + "'");
		return same;
	}

	private boolean containsText(String haystack, String needle, String label) {
		String h = normalizeText(haystack);
		String n = normalizeText(needle);
		boolean ok = h.contains(n);
		if (!ok) logger.debug("[未包含] " + label + " | 期望包含='" + needle + "' 但實際為='" + haystack + "'");
		return ok;
	}

	private boolean eqNumber(String s2, String s3, String label) {
		String a = normalizeNumber(s2);
		String b = normalizeNumber(s3);
		boolean same = a.equals(b);
		if (!same) logger.debug("[數字不一致] " + label + " | S2='" + s2 + "'(" + a + ") vs S3='" + s3 + "'(" + b + ")");
		return same;
	}

	private String normalizeText(String s) {
		if (s == null) return "";
		return s.replace('\u00A0',' ')
				.replace('\u3000',' ')
				.replaceAll("\\s+"," ")
				.trim();
	}

	private String normalizeNumber(String s) {
		if (s == null) return "";
		String t = s.replace(",", "").trim();
		if (t.contains("%")) t = t.replaceAll("[^0-9.%]", "");
		else t = t.replaceAll("[^0-9.]", "");
		if (t.endsWith(".")) t = t.substring(0, t.length() - 1);
		return t;
	}

	private List<String> collectCheckedTextsFromTable(String tableId, int colIndex) {
		List<String> out = new ArrayList<>();
		try {
			List<WebElement> rows = findElements(By.cssSelector("#" + tableId + " tbody tr"), 2000);
			for (WebElement tr : rows) {
				try {
					WebElement chk = tr.findElement(By.cssSelector("input[type='checkbox']"));
					if (chk != null && chk.isSelected()) {
						List<WebElement> tds = tr.findElements(By.tagName("td"));
						if (tds.size() >= colIndex) out.add(safeText(tds.get(colIndex - 1)));
					}
				} catch (Exception ignore) {}
			}
		} catch (Exception e) {
			logger.debug("collectCheckedTextsFromTable(" + tableId + ") 例外: " + e.getMessage());
		}
		return out;
	}

	private boolean verifyTableTitleLoose(List<String> expected, String thXpath) {
		try {
			List<WebElement> ths = findElements(By.xpath(thXpath), 5000);
			List<String> actual = new ArrayList<>();
			for (WebElement th : ths) actual.add(norm(th.getText()));
			for (String e : expected) {
				String ne = norm(e);
				boolean ok = actual.stream().anyMatch(a -> a.contains(ne) || ne.contains(a));
				if (!ok) {
					logger.debug("[Title not found] exp='" + e + "' actual=" + actual);
					return false;
				}
			}
			return true;
		} catch (Exception ex) {
			logger.debug("verifyTableTitleLoose error: " + ex.getMessage());
			return false;
		}
	}

	private String norm(String s){ if (s==null) return ""; return s.replace('\u00A0',' ').replace('\u3000',' ').replaceAll("\\s+","").trim(); }

	private WebElement findRowByPolicyNoLoose() {
		try {
			String xp = "(//table[contains(@class,'search-table') and (contains(@id,'tran') or contains(@id,'list'))]"
					  + "//tr[.//td[contains(normalize-space(),'" + POLICY_NO + "')]])[1]";
			return findElement(By.xpath(xp), false, 2000);
		} catch (Exception e) { return null; }
	}

	private String getWholePageText() {
		try { WebElement body = driver.findElement(By.tagName("body")); return body.getText() == null ? "" : body.getText(); }
		catch (Exception e) { return ""; }
	}

	private String tryGetText(By by) {
		try { WebElement el = findElement(by, false, 1500); return el == null ? "" : safeText(el); }
		catch (Exception e) { return ""; }
	}

	/* =========================
	   Step3 定位輔助
	   ========================= */
	private boolean locateStep3Root(String tag) {
		waitForWindowHandlesToSettle(700, 12000);
		switchToMostRecentAliveWindow();

		if (switchToFrameContaining(By.id("loanData3"), 4000)) return true;
		if (switchToFrameContaining(By.id("tdclinetName"), 4000)) return true;
		if (switchToFrameContaining(By.id("tdTtransfer"), 4000)) return true;

		List<String> order = new ArrayList<>(driver.getWindowHandles());
		for (int i = order.size() - 1; i >= 0; i--) {
			try {
				driver.switchTo().window(order.get(i));
				try { Thread.sleep(160); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
				if (switchToFrameContaining(By.id("loanData3"), 1800)) return true;
				if (switchToFrameContaining(By.id("tdclinetName"), 1800)) return true;
				if (switchToFrameContaining(By.id("tdTtransfer"), 1800)) return true;
			} catch (Exception ignore) {}
		}
		logger.debug("[locateStep3Root] 無法在任何 window/iframe 找到 Step3 關鍵元素 (tag=" + tag + ")");
		return false;
	}

	private boolean switchToFrameContaining(By by, int timeoutMs) {
		long end = System.currentTimeMillis() + timeoutMs;
		while (System.currentTimeMillis() < end) {
			try {
				driver.switchTo().defaultContent();
				WebElement el = findElement(by, false, 300);
				if (el != null) return true;

				List<WebElement> ifs = driver.findElements(By.tagName("iframe"));
				for (int i = 0; i < ifs.size(); i++) {
					try {
						driver.switchTo().frame(i);
						el = findElement(by, false, 250);
						if (el != null) return true;
					} catch (Exception ignore) {
					} finally {
						driver.switchTo().defaultContent();
					}
				}
			} catch (Exception ignore) {}
			try { Thread.sleep(120); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
		}
		try { driver.switchTo().defaultContent(); } catch (Exception ignore) {}
		return false;
	}

	// =========================
	// 「滾動脈衝」— 觸發 Step3 懶載/啟用底部按鈕
	// =========================
	private void pulseScrollGloballyAndTopIframes() {
		// 当前 window 的 defaultContent
		pulseScrollCurrentContext();
		// 每個 top-level iframe
		try {
			driver.switchTo().defaultContent();
			List<WebElement> ifs = driver.findElements(By.tagName("iframe"));
			for (int i = 0; i < ifs.size(); i++) {
				try {
					driver.switchTo().frame(i);
					pulseScrollCurrentContext();
					driver.switchTo().defaultContent();
				} catch (Exception e) {
					driver.switchTo().defaultContent();
				}
			}
		} catch (Exception ignore) {}
		try { driver.switchTo().defaultContent(); } catch (Exception ignore) {}
	}

	private void pulseScrollCurrentContext() {
		JavascriptExecutor js = (JavascriptExecutor) driver;
		try {
			// 先到頂、再到底，刺激 onscroll
			js.executeScript("try{window.scrollTo(0,0);}catch(e){}");
			js.executeScript("try{window.scrollTo(0,document.body.scrollHeight);}catch(e){}");
		} catch (Exception ignore) {}
		// 滾動所有可滾容器（overflow:auto/scroll 且可滾）
		try {
			String script =
				"try{var c=[document.scrollingElement||document.documentElement, document.body];"
			  + "var list=Array.from(document.querySelectorAll('*')).filter(function(e){"
			  + "  try{var s=getComputedStyle(e); return (s.overflowY==='auto'||s.overflowY==='scroll')&&e.scrollHeight>e.clientHeight+50;}catch(err){return false;}"
			  + "});"
			  + "c=c.concat(list);"
			  + "c.forEach(function(e){try{e.scrollTop=0;}catch(err){}});"
			  + "c.forEach(function(e){try{e.scrollTop=e.scrollHeight;}catch(err){}});"
			  + "return c.length;}catch(ex){return 0;}";
			js.executeScript(script);
		} catch (Exception ignore) {}
	}

	// =========================
	// NEW: jQuery/DOM 穩定等待
	// =========================
	private void waitForStable() {
		try {
			new WebDriverWait(driver, Duration.ofSeconds(15)).until(d -> {
				try {
					JavascriptExecutor js = (JavascriptExecutor) d;
					Object ready = js.executeScript("return document.readyState");
					Object jq = js.executeScript("return window.jQuery ? jQuery.active : 0");
					boolean docReady = "complete".equals(ready);
					boolean ajaxIdle = !(jq instanceof Long) || ((Long) jq) == 0L;
					return docReady && ajaxIdle;
				} catch (Exception e) { return true; }
			});
			Thread.sleep(200);
		} catch (InterruptedException ie) {
			Thread.currentThread().interrupt();
		} catch (Exception ignore) {}
	}

	// =========================
	// NEW: 勾全文件/提醒（若存在）
	// =========================
	private void checkAllIfExists(By by) {
		try {
			List<WebElement> cbs = findElements(by, 1500);
			for (WebElement cb : cbs) {
				try {
					if (cb.isEnabled() && !cb.isSelected()) {
						((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView({block:'center'});", cb);
						cb.click();
					}
				} catch (Exception ignore) {}
			}
		} catch (Exception ignore) {}
	}

	// =========================
	// NEW: 動態 rowId 與相關定位器
	// =========================
	private String getRowId() {
		if (detectedRowId == null || detectedRowId.isBlank()) {
			detectedRowId = detectRowId();
			if (detectedRowId == null || detectedRowId.isBlank()) detectedRowId = POLICY_NO; // 保底
		}
		return detectedRowId;
	}
	private void setRowId(String r) { detectedRowId = r; }

	/** 偵測 Step2 下第一列 r{rowId}，回傳 rowId（例：80241059） */
	private String detectRowId() {
		try {
			List<WebElement> rows = findElements(By.cssSelector("#divInputChg table.search-table tbody tr[id^='r']"), 2000);
			if (rows != null && !rows.isEmpty()) {
				String id = rows.get(0).getAttribute("id"); // r80241059
				if (id != null) return id.replaceFirst("^r", "");
			}
		} catch (Exception ignore) {}
		return null;
	}

	// 動態欄位定位器
	private By loanTypeSelectBy() { return By.id("lt" + getRowId()); }
	private By loanReasonSelectBy() { return By.id("lr" + getRowId()); }
	private By currentLoanAmountInputBy() { return By.id("cl" + getRowId()); }

	private String policyChangeTableDataXpath() {
		return "(//div[@id='divInputChg']//table[@width='100%' and @class='table table-bordered search-table']/tbody/tr[@id='r" + getRowId() + "'])[1]/td[not(contains(@class,'pc_hide'))]";
	}
	private String enterLoanContentTableDataXpath() {
		return "(//div[@id='divInputChg']//table[@width='100%' and @class='table table-bordered search-table' and @id='loanData2']/tbody/tr[@id='r" + getRowId() + "'])[1]/td";
	}
	private String terminateContractStep2ChangePolicyTableDataXpath() {
		return "(//div[@id='divInputChg']//table[@width='100%' and @class='table table-bordered search-table']/tbody/tr[@id='r" + getRowId() + "'])[1]/td[not(contains(@class,'pc_hide'))]";
	}

	/* =========================
	   NEW: 短命視窗（下載泡泡）處理工具
	   ========================= */
	private String waitAnyNewWindow(Set<String> before, long timeoutMs) {
		long end = System.currentTimeMillis() + timeoutMs;
		while (System.currentTimeMillis() < end) {
			try {
				Set<String> now = driver.getWindowHandles();
				if (now.size() > before.size()) {
					for (String h : now) if (!before.contains(h)) return h;
				}
				Thread.sleep(120);
			} catch (Exception ignore) {
				try { Thread.sleep(120); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
			}
		}
		return null;
	}

	private void waitWindowClosed(String handle, long timeoutMs) {
		long end = System.currentTimeMillis() + timeoutMs;
		while (System.currentTimeMillis() < end) {
			try {
				if (!driver.getWindowHandles().contains(handle)) return;
			} catch (Exception ignore) { return; }
			try { Thread.sleep(150); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
		}
	}

	private void refocusMainWindow() {
		try {
			Set<String> handles = driver.getWindowHandles();
			if (step1Handle != null && handles.contains(step1Handle)) {
				driver.switchTo().window(step1Handle);
			} else {
				switchToMostRecentAliveWindow();
			}
			driver.switchTo().defaultContent();
		} catch (Exception ignore) {}
	}

	private boolean clickAndHandleShortLivedWindow(WebElement el, long newHandleWaitMs, long autoCloseWaitMs) {
		try {
			Set<String> before = driver.getWindowHandles();
			try { ((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView({block:'center'});", el); } catch (Exception ignore) {}
			try { click(el, true); } catch (Exception e) { ((JavascriptExecutor) driver).executeScript("arguments[0].click();", el); }

			String newH = waitAnyNewWindow(before, newHandleWaitMs);
			if (newH != null) {
				// 不在小窗裡操作，僅等待它自動關閉
				try { driver.switchTo().window(newH); } catch (Exception ignore) {}
				waitWindowClosed(newH, autoCloseWaitMs);
			}
			waitForWindowHandlesToSettle(700, 12000);
			refocusMainWindow();
			return true;
		} catch (Exception e) {
			return false;
		}
	}
}

<!--
**fontevrand/fontevrand** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
