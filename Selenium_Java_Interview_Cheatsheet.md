# Selenium with Java — Associate-Level Interview Cheatsheet

---

## 1\. Selenium Basics

**What is Selenium?** Open-source tool/framework for automating web browsers. Used for functional testing, regression testing, and web scraping.

**Selenium Suite Components:** | Component | Purpose | |---|---| | Selenium IDE | Record-and-playback browser extension | | Selenium WebDriver | Core API to control browsers programmatically | | Selenium Grid | Run tests on multiple machines/browsers in parallel | | Selenium RC (deprecated) | Old remote control tool, replaced by WebDriver |

**Why Selenium?**

- Open source, free  
- Supports multiple languages (Java, Python, C\#, JS, Ruby)  
- Supports multiple browsers (Chrome, Firefox, Edge, Safari)  
- Supports multiple OS (Windows, Linux, Mac)  
- Integrates with TestNG/JUnit, Maven, Jenkins, Docker for CI/CD

**Limitations of Selenium:**

- Cannot test mobile native apps (use Appium)  
- Cannot handle CAPTCHA, OTP, Windows-native popups (use tools like Sikuli/AutoIt)  
- No built-in reporting (need Extent Reports/Allure)  
- No built-in test management (need TestNG/JUnit)

---

## 2\. Architecture — How WebDriver Works

```
Test Script (Java) → JSON Wire Protocol / W3C Protocol → Browser Driver (chromedriver.exe) → Browser
```

- Selenium WebDriver uses the **W3C WebDriver Protocol** (older versions used JSON Wire Protocol).  
- Each browser has its own driver executable:  
  - Chrome → `chromedriver`  
  - Firefox → `geckodriver`  
  - Edge → `msedgedriver`  
  - Safari → `safaridriver`  
- The driver acts as a bridge, translating Selenium commands into browser-specific instructions via HTTP.

---

## 3\. Setting Up a Project

**Maven Dependency (pom.xml):**

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.21.0</version>
</dependency>
```

**Driver Management:** Since Selenium 4.6+, **Selenium Manager** auto-downloads the correct driver binary — no need to manually manage `chromedriver.exe` or use WebDriverManager (though WebDriverManager is still common in older codebases).

```java
WebDriver driver = new ChromeDriver(); // Selenium Manager handles driver binary automatically
```

---

## 4\. Launching Browsers

```java
WebDriver driver = new ChromeDriver();
WebDriver driver = new FirefoxDriver();
WebDriver driver = new EdgeDriver();
```

**With Options (headless, arguments, etc.):**

```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless=new");
options.addArguments("--start-maximized");
options.addArguments("--disable-notifications");
WebDriver driver = new ChromeDriver(options);
```

---

## 5\. Locators (Finding Elements)

| Locator | Example |
| :---- | :---- |
| ID | `driver.findElement(By.id("username"))` |
| Name | `driver.findElement(By.name("email"))` |
| ClassName | `driver.findElement(By.className("btn-primary"))` |
| TagName | `driver.findElement(By.tagName("input"))` |
| LinkText | `driver.findElement(By.linkText("Sign Up"))` |
| PartialLinkText | `driver.findElement(By.partialLinkText("Sign"))` |
| CSS Selector | `driver.findElement(By.cssSelector("input[name='q']"))` |
| XPath | `driver.findElement(By.xpath("//input[@id='search']"))` |

**Priority Order (interview favorite):** ID \> Name \> CSS Selector \> XPath (ID is fastest & most reliable; XPath is slowest but most flexible).

### XPath Cheatsheet

```
//tagname[@attribute='value']          → absolute match
//*[contains(@class,'btn')]            → partial match
//div[text()='Login']                  → exact text match
//input[starts-with(@id,'user')]       → starts-with
//div[@id='parent']//span              → descendant
//div/following-sibling::span
//div/preceding-sibling::span
//div/parent::node()
//input[@type='text' and @name='q']    → AND condition
```

**Absolute vs Relative XPath:**

- Absolute: starts with `/html/...` — fragile, breaks with DOM changes.  
- Relative: starts with `//` — preferred, more resilient.

### CSS Selector Cheatsheet

```
#id                     → id selector
.class                  → class selector
tag[attr='value']       → attribute selector
tag[attr^='value']      → starts with
tag[attr$='value']      → ends with
tag[attr*='value']      → contains
div > span              → direct child
div span                → descendant
```

**findElement vs findElements:**

- `findElement()` → returns first matching `WebElement`, throws `NoSuchElementException` if not found.  
- `findElements()` → returns `List<WebElement>`, returns empty list if none found (no exception).

---

## 6\. WebDriver Commands (Browser & Navigation)

```java
driver.get("https://example.com");        // opens URL
driver.navigate().to("https://example.com");
driver.navigate().back();
driver.navigate().forward();
driver.navigate().refresh();

driver.getTitle();
driver.getCurrentUrl();
driver.getPageSource();

driver.close();   // closes current window/tab
driver.quit();    // closes all windows and ends the WebDriver session
```

**close() vs quit():** `close()` shuts only the active window; `quit()` terminates the entire browser session and releases resources.

---

## 7\. Element Interactions

```java
element.click();
element.sendKeys("text");
element.clear();
element.getText();
element.getAttribute("value");
element.getCssValue("color");
element.isDisplayed();
element.isEnabled();
element.isSelected();
element.submit();
```

---

## 8\. Waits (Synchronization) — Very Important Topic

**Why needed?** Modern web apps load elements asynchronously (AJAX/JS); waits prevent `NoSuchElementException` / `ElementNotInteractableException`.

### Implicit Wait

Applies globally for the whole driver session; polls DOM for a set time before throwing exception.

```java
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

### Explicit Wait

Waits for a specific condition on a specific element.

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("submit")));
wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("submit")));
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.id("loader")));
wait.until(ExpectedConditions.alertIsPresent());
```

### Fluent Wait

Like explicit wait but with custom polling frequency and exceptions to ignore.

```java
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
        .withTimeout(Duration.ofSeconds(20))
        .pollingEvery(Duration.ofSeconds(2))
        .ignoring(NoSuchElementException.class);

WebElement element = fluentWait.until(d -> d.findElement(By.id("submit")));
```

**Best Practice:** Never mix implicit wait \+ explicit wait in the same script (can cause unpredictable wait times).

**Interview Tip:** Explain difference clearly: | Wait Type | Scope | Use Case | |---|---|---| | Implicit | Global (all elements) | Simple, general delay | | Explicit | Specific element/condition | Most common, recommended | | Fluent | Specific element \+ custom polling | Advanced control |

---

## 9\. Handling Dropdowns

```java
Select select = new Select(driver.findElement(By.id("country")));
select.selectByVisibleText("India");
select.selectByValue("IN");
select.selectByIndex(2);

select.getOptions();          // list all options
select.getFirstSelectedOption();
select.deselectAll();         // only for multi-select
```

---

## 10\. Handling Alerts / Pop-ups

```java
Alert alert = driver.switchTo().alert();
alert.accept();
alert.dismiss();
alert.getText();
alert.sendKeys("some text");  // for prompt alerts
```

Types: **Simple Alert**, **Confirmation Alert**, **Prompt Alert**.

---

## 11\. Handling Multiple Windows / Tabs

```java
String parentWindow = driver.getWindowHandle();
Set<String> allWindows = driver.getWindowHandles();

for (String window : allWindows) {
    if (!window.equals(parentWindow)) {
        driver.switchTo().window(window);
    }
}
driver.switchTo().window(parentWindow); // switch back
```

---

## 12\. Handling Frames (iframe)

```java
driver.switchTo().frame("frameName");     // by name/id
driver.switchTo().frame(0);               // by index
driver.switchTo().frame(webElement);      // by WebElement
driver.switchTo().defaultContent();       // back to main page
driver.switchTo().parentFrame();          // back to parent frame
```

---

## 13\. Actions Class (Mouse & Keyboard Actions)

```java
Actions actions = new Actions(driver);

actions.moveToElement(element).perform();               // hover
actions.clickAndHold(element).perform();
actions.dragAndDrop(source, target).perform();
actions.doubleClick(element).perform();
actions.contextClick(element).perform();                // right-click
actions.keyDown(Keys.CONTROL).click(element).keyUp(Keys.CONTROL).perform(); // ctrl+click
actions.sendKeys(Keys.ENTER).perform();
```

---

## 14\. JavaScriptExecutor (When Selenium Native Methods Fail)

```java
JavascriptExecutor js = (JavascriptExecutor) driver;

js.executeScript("arguments[0].click();", element);              // JS click
js.executeScript("arguments[0].scrollIntoView(true);", element); // scroll
js.executeScript("window.scrollBy(0,500)");
js.executeScript("arguments[0].value='text';", element);         // set value
String title = (String) js.executeScript("return document.title;");
```

---

## 15\. Handling File Upload / Download

**Upload (if `<input type="file">` exists):**

```java
driver.findElement(By.id("fileUpload")).sendKeys("C:\\path\\to\\file.txt");
```

(No need for a dialog — Selenium can't interact with OS-level dialogs directly, so `sendKeys` on the input element is the standard workaround.)

**Download:** Configure browser preferences to auto-download to a folder (via `ChromeOptions` \+ `Map` of preferences) since Selenium can't control OS save dialogs.

---

## 16\. Screenshots

```java
File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(screenshot, new File("screenshot.png")); // needs commons-io
```

---

## 17\. Cookies

```java
driver.manage().addCookie(new Cookie("key", "value"));
driver.manage().getCookies();
driver.manage().getCookieNamed("key");
driver.manage().deleteCookieNamed("key");
driver.manage().deleteAllCookies();
```

---

## 18\. Exceptions (Commonly Asked)

| Exception | Cause |
| :---- | :---- |
| `NoSuchElementException` | Element not found in DOM |
| `ElementNotInteractableException` | Element present but not interactable (hidden/disabled) |
| `StaleElementReferenceException` | Element reference is outdated (DOM refreshed/re-rendered) |
| `TimeoutException` | Wait condition not met within given time |
| `NoAlertPresentException` | Trying to switch to alert when none exists |
| `InvalidSelectorException` | Malformed locator syntax |
| `SessionNotCreatedException` | Driver/browser version mismatch |

**How to handle StaleElementReferenceException:** Re-locate the element after DOM changes; wrap in retry logic or explicit wait.

---

## 19\. Page Object Model (POM) — Design Pattern

**Concept:** Each web page is represented as a class; web elements are fields, and page actions are methods. Separates test logic from UI locators → improves maintainability.

```java
public class LoginPage {
    WebDriver driver;

    @FindBy(id = "username")
    WebElement username;

    @FindBy(id = "password")
    WebElement password;

    @FindBy(id = "loginBtn")
    WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void login(String user, String pass) {
        username.sendKeys(user);
        password.sendKeys(pass);
        loginButton.click();
    }
}
```

**PageFactory:** Utility class that initializes `@FindBy` annotated `WebElement`s lazily (elements are located only when used, using dynamic proxies).

**Benefits of POM:**

- Reusability of page classes across test cases  
- Easier maintenance (locator changes in one place)  
- Improved readability

---

## 20\. TestNG (Most Common Framework Paired with Selenium)

```java
import org.testng.annotations.*;

public class LoginTest {

    @BeforeSuite
    public void beforeSuite() { }

    @BeforeClass
    public void setup() { /* launch browser */ }

    @BeforeMethod
    public void beforeEachTest() { }

    @Test(priority = 1)
    public void testLogin() { }

    @Test(priority = 2, dependsOnMethods = "testLogin")
    public void testDashboard() { }

    @AfterMethod
    public void afterEachTest() { }

    @AfterClass
    public void teardown() { /* close browser */ }

    @AfterSuite
    public void afterSuite() { }
}
```

**Annotation Execution Order:** `@BeforeSuite → @BeforeClass → @BeforeMethod → @Test → @AfterMethod → @AfterClass → @AfterSuite`

**Key TestNG Features:**

- `@DataProvider` — for data-driven testing  
- `@Test(priority=)` — control execution order  
- `@Test(dependsOnMethods=)` — dependency between tests  
- `testng.xml` — suite configuration file (parallel execution, grouping tests)  
- Assertions: `Assert.assertEquals()`, `Assert.assertTrue()`, `Assert.assertFalse()`

```java
@DataProvider(name = "loginData")
public Object[][] getData() {
    return new Object[][] {
        {"user1", "pass1"},
        {"user2", "pass2"}
    };
}

@Test(dataProvider = "loginData")
public void testLogin(String user, String pass) { }
```

---

## 21\. Selenium Grid

**Purpose:** Distributed test execution — run tests on multiple machines/browsers/OS in parallel.

**Architecture (Selenium 4):**

- **Hub** — central point that receives test requests  
- **Node** — machine that executes tests on a specific browser/OS

```java
WebDriver driver = new RemoteWebDriver(new URL("http://hub-ip:4444/wd/hub"), options);
```

Used with Docker (`docker-compose` for grid setup) commonly in CI/CD pipelines.

---

## 22\. Selenium 4 Key New Features (Frequently Asked)

- Native **W3C WebDriver Protocol** support (no more JSON Wire Protocol conversion)  
- **Relative Locators**: `above()`, `below()`, `toLeftOf()`, `toRightOf()`, `near()`

```java
driver.findElement(RelativeLocator.with(By.tagName("input")).above(By.id("submit")));
```

- **Selenium Manager** — auto driver management, no manual binary downloads  
- Improved **Selenium Grid** with better observability (visual debugging)  
- **Chrome DevTools Protocol (CDP)** integration — network interception, console logs, geolocation  
- New window/tab creation:

```java
driver.switchTo().newWindow(WindowType.TAB);
driver.switchTo().newWindow(WindowType.WINDOW);
```

---

## 23\. Framework Concepts (Often Asked at Associate Level)

**Types of Automation Frameworks:** | Framework | Description | |---|---| | Linear/Record-playback | Simple script per test, no reusability | | Modular | Breaks app into modules, each with its own script | | Data-Driven | Test logic separated from test data (Excel/CSV/DB) | | Keyword-Driven | Actions represented as keywords in external files | | Hybrid | Combination of above | | BDD (Behavior Driven Development) | Uses Gherkin (Given-When-Then) syntax, e.g. Cucumber |

**Data-Driven Testing with Apache POI (Excel):**

```java
FileInputStream fis = new FileInputStream("data.xlsx");
Workbook workbook = new XSSFWorkbook(fis);
Sheet sheet = workbook.getSheet("Sheet1");
Row row = sheet.getRow(1);
String value = row.getCell(0).getStringCellValue();
```

---

## 24\. CI/CD Integration Basics

- **Maven** — build & dependency management (`pom.xml`)  
- **Jenkins** — schedules/triggers automated test runs  
- **Git/GitHub** — version control for test scripts  
- **Docker** — containerized browser/grid environments  
- Typical flow: Code commit → Jenkins triggers Maven build → Runs TestNG suite → Publishes reports (Extent Report/Allure)

---

## 25\. Best Practices (Good to Mention in Interviews)

1. Always prefer **explicit waits** over `Thread.sleep()`.  
2. Use **Page Object Model** for maintainability.  
3. Keep locators in one place; prefer **CSS/ID** over XPath for performance.  
4. Use **relative XPath**, avoid absolute XPath.  
5. Implement **try-catch / retry logic** for flaky elements (`StaleElementReferenceException`).  
6. Use **data-driven testing** to avoid hardcoding test data.  
7. Run tests in **headless mode** for CI/CD speed.  
8. Take **screenshots on failure** for debugging (use TestNG's `ITestListener`).  
9. Clean up with `driver.quit()` in `@AfterMethod`/`@AfterClass` to avoid orphan browser processes.  
10. Avoid `Thread.sleep()` — it slows down suites and is not condition-based.

---

## 26\. Quick-Fire Q\&A (Rapid Revision)

**Q: Difference between Selenium 3 and Selenium 4?** A: Selenium 4 uses native W3C protocol, has relative locators, built-in Selenium Manager, improved Grid UI, CDP integration, new window/tab API.

**Q: Difference between `driver.get()` and `driver.navigate().to()`?** A: `get()` loads a new page and waits for it to load; `navigate().to()` does the same but maintains browser history (supports back/forward navigation more explicitly). Functionally very similar in modern Selenium.

**Q: How do you handle dynamic elements (e.g., changing IDs)?** A: Use relative/partial XPath (`contains()`, `starts-with()`), CSS attribute selectors, or relative locators.

**Q: What is the difference between `Thread.sleep()` and `WebDriverWait`?** A: `Thread.sleep()` is a static, fixed wait regardless of whether the condition is met — wastes time or isn't enough. `WebDriverWait` polls until condition is true or timeout — dynamic & efficient.

**Q: How do you run tests in parallel?** A: Configure `testng.xml` with `parallel="methods"/"classes"/"tests"` and `thread-count`, or use Selenium Grid across multiple nodes.

**Q: What is a Headless browser?** A: A browser running without a GUI — faster execution, commonly used in CI/CD pipelines.

```java
options.addArguments("--headless=new");
```

**Q: How to handle a StaleElementReferenceException?** A: Re-find the element right before interacting with it, or wrap interaction in a retry loop with explicit wait.

**Q: What's the difference between `@FindBy` and `findElement()`?** A: `@FindBy` is a POM/PageFactory annotation for lazy element initialization; `findElement()` is an immediate WebDriver API call.

**Q: How does Selenium know an element is "ready" to interact with?** A: It doesn't automatically — that's why waits (implicit/explicit/fluent) exist; Selenium itself has no built-in AJAX awareness.

---

## 27\. Sample End-to-End Test Script

```java
import org.openqa.selenium.*;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;

public class LoginTest {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.get("https://example.com/login");

        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        WebElement username = wait.until(
            ExpectedConditions.visibilityOfElementLocated(By.id("username")));
        username.sendKeys("testuser");

        driver.findElement(By.id("password")).sendKeys("password123");
        driver.findElement(By.id("loginBtn")).click();

        wait.until(ExpectedConditions.urlContains("dashboard"));
        System.out.println("Login successful. Title: " + driver.getTitle());

        driver.quit();
    }
}
```

---

## 28\. One-Page Summary Table

| Category | Key Classes/Methods |
| :---- | :---- |
| Setup | `ChromeDriver`, `ChromeOptions`, Selenium Manager |
| Locators | `By.id/name/className/cssSelector/xpath` |
| Waits | `WebDriverWait`, `FluentWait`, `implicitlyWait()` |
| Actions | `Actions` class — hover, drag-drop, right-click |
| Alerts | `driver.switchTo().alert()` |
| Frames | `driver.switchTo().frame()` |
| Windows | `getWindowHandles()`, `switchTo().window()` |
| JS | `JavascriptExecutor` |
| Design Pattern | Page Object Model \+ PageFactory |
| Test Framework | TestNG (`@Test`, `@DataProvider`, `testng.xml`) |
| Parallel Execution | Selenium Grid (Hub-Node), TestNG parallel |
| Exceptions | `NoSuchElementException`, `StaleElementReferenceException`, `TimeoutException` |

---

*End of Cheatsheet — Good luck with your interview\!*  
