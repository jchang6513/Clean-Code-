# Chapter 3: Functions

## Small!
撰寫 function 的第一個要求就是要夠小，當 function 行數過多會嚴重的影響到閱讀。至於 function 到底要多小，作者建議是在 20 - 30 行內

```typescript
// FUNC 3.1
// if statement 內的 expression 還是過多了
const renderPageWithSetupsAndTeardowns = (
  pageData: PageData, 
  isSuite: boolean
): string => {
  const isTestPage = pageData.hasAttribute('Test'); 
  if (isTestPage) {
      const testPage = pageData.getWikiPage(); 
      const newPageContent = new StringBuffer();       
      includeSetupPages(testPage, newPageContent, isSuite);     
      newPageContent.append(pageData.getContent()); 
      includeTeardownPages(testPage, newPageContent, isSuite); 
      pageData.setContent(newPageContent.toString());
  }
  return pageData.getHtml(); 
}

// FUNC 3.2
// 將內容簡化為一個 function call 會有一個更舒服的閱讀體驗
const renderPageWithSetupsAndTeardowns = (
  pageData: PageData, 
  isSuite: boolean
): string => { 
  if (isTestPage(pageData)) {
      includeSetupAndTeardownPages(pageData, isSuite); 
  }
  return pageData.getHtml();
}
```
### Blocks and Indenting
這裡主要想說的是在任一 statement 區塊 (*if*, *else*, *while*) 的程式碼都建議簡化為一行表示。而一行程式碼最好是一個 function call，這樣除了可以大幅的減少原function 的長度外也可以用一個有意義的 function name 簡單的描述該區塊所要做的工作。

當我們將 function 瘦身的同時，我們也限制了巢狀結構的增長，可以將縮排數量限制在兩層縮排內，有效的增加 function 的可讀性。

## Do One Thing
> **每個 function 只要做一件事就好，並且要做對。**

但問題是我們要怎麼定義**一件事**？作者提出了一個做法，當我們能夠用一個 ***TO* paragraph** 描述這個 function 則代表他只做了一件事，換成中文的話我認為可以用**當**來描述(中譯本是說*要*)。若我們以`FUNC 3.2` 為例，我們可以說

> 當 renderPageWithSetupsAndTeardowns 時，我們會檢查他是不是一個測試頁面，如過他是測試頁我們會 include setups and teardowns，但無論如何我們都會回傳一個 HTML 頁

🤔: 我覺得這樣講挺話術的

我認為最明確判斷最一件事的方式是書中有的另一個說法，function 只能做到 function name 描述的**抽象概念**。我們可以用 `FUNC 3.1` 作為參考，`FUNC 3.1` 除了 render page 外，也做了 include setup page 和 include teardown page，而後面兩個抽象的概念是可以被抽出至一個獨立 function 的。

所以粗暴一點的觀察 function 是否只做**一件事**的方式，可以用是否能夠從 function 內抽出新 function 來判斷。

## One Level of Abstraction per Function 
### Reading Code from Top to Bottom: The Stepdown Rule

## Switch Statements 
沒什麼救的東西，只要有 switch statement，function 就一定會太大然後也很難只做一件事。能做的就是多利用 polymorphism 處理。

``` typescript
// FUNC 3.3

const calculatePay = (e: Employee ): Money => {
  switch (e.type) { 
    case COMMISSIONED:
      return calculateCommissionedPay(e); 
    case HOURLY:
      return calculateHourlyPay(e); 
    case SALARIED:
      return calculateSalariedPay(e); 
    default:
      throw new InvalidEmployeeType(e.type); 
  }
}
```

以 `FUNC 3.3` 為例，除了 function 過大且做了不只一件事以外，也違反了多個OO設計的原則（書中提到的SRP 和 OCP），但最主要的問題是這類型的 switch statement 將有很大的機會出現在各種 function 中 (ex: `isPayday(e: Employee, date: Date)`, `deliverPay(e: Employee, date: Date)`)

作者對此提出的解法是將 switch statement 寫在 Abstract Factory 中，Factory 可以產生對應的 instance 且這些 instance 會繼承相同的 Employee 介面

``` typescript
class Employee {
  isPayday(): boolean { 
    //...
  }
  
  calculatePay(): Money {
    //...
  }
  
  deliverPay(pay: Money) {
    //...
  }
}

-----------------
  
class EmployeeFactory {
  static makeEmployee(r: EmployeeRecord): Employee {
    case COMMISSIONED:
      return new CommissionedEmployee(r) ;
    case HOURLY:
      return new HourlyEmployee(r);
    case SALARIED:
      return new SalariedEmploye(r);
    default:
      return new InvalidEmployee(r);  
  }
}
```

## Use Descriptive Names
原則上就是要取一個讓你一看到就能清楚了解 function 實際功能的名稱。幾個可以注意的內容

- 不要怕名稱太長。描寫清楚但很長的名稱勝過精簡但語意不清的名稱
- 不要覺得取名浪費時間。清楚的名稱對未來的開發或重構有一定的優勢
- 維持一致的命名邏輯。

## Function Arguments
參數是一個難題，沒有參數(niladic)是最理想的 function，最不濟也要盡量限制在三個參數以內。參數的多寡不僅會提高程式碼閱讀的難度，也會增加測試的複雜度。

### Common Monadic (one argument) Forms
一些常見的單一參數 function
``` typescript
type fileExists = (file: File) => boolean;
type fileOpen = (file: File) => InputStream;
type passwordAttemptFailedNtimes = (attempts: number) => void;
```

避免使用會改變參數的值的回傳結果，盡量讓 function 把結果作為回傳值傳出
```typescript
// instead of declare this type of function
type includeSetupPageInto = (pageText: StringBuffer) => void;

// this type of function will be better
type includeSetupPageInto = (pageText: StringBuffer) => StringBuffer;
```
### Flag Arguments
將一個 boolean 作為 flag 參數傳入 function 是一個不理想的做法，這讓 function 會因為 flag 的變化做了不只**一件事**

### Dyadic (two argument) Functions
兩個參數並不是大問題，但如果有方式可以讓參數變少，那就應該利用。
- 必然的兩個參數
``` typescript
type createPoint: (x: number, y: number) => Point;
```

- 可以簡化的狀況
``` typescript
type write(file: File, content: string) => void;
```
file 和 content 兩者沒有一個自然的排列順序，下面的做法會更好
``` typescript
interface File {
  write: (content: string) => void;
}
```

### Triads 
沒什麼好說的，就盡量避免

### Argument Objects
用類別包裝參數是一個減少參數數量的方法，以下面的 function 為例，
``` typescript
type makeCircle = (x: number, y: number, rad: number) => Circle;
```
我們要在卡氏座標上做一個圓我們需要知道座標位置 x, y 和圓的半徑，這邊與其傳遞三個變數，我們可以用一個 `Point` 類別封裝座標位置，這樣就可以有效的減少參數的數量，讓程式碼更為簡潔
``` typescript
type makeCircle = (center: Point, rad: number) => Circle;
```

### Argument Lists 
有時會有傳遞不同數量的參數的需求，例如以下 function
``` typescript
type sum = (...numbers: number[]) => number;
```
此 function 會將使用者傳入的所以參數進行加總，因為這些參數都是被相同的對待，因此可以視為 List 型態的一個參數，所以這個 function 是一個 monadic forms。但同樣的這種寫法也要避免設計超過三個參數需求的 function。

### Verbs and Keywords
function 的命名要配合參數掌握其動詞和關鍵字，例如 `write(name)`，我們可以很清楚的知道 name 會被寫入，更好的就是將寫入的東西寫出來，像是 `writeField(name)`。

而像是 `toHaveBeenCalled(nthCall, ...args)` 這樣的 function，如果改為`toHaveBeenNthCalledWith(nthCall, ...args)` 可以更清楚的描述 function 功能並減少記憶參數順序的負擔。

## Have No Side Effects
不要讓 function 有任何的 side effect，side effect 不只破壞做一件事的準則，同時也會造成不可預期的改變。如果是不可避免的情況則要在 function name 中讓人一目了然，例如以下的 function
```typescript
class UserValidator {
  private cryptographer: Cryptographer;
  
  checkPassword(userName: string, password: string) { 
    const user = UserGateway.findByName(userName);
    if (user != User.NULL) {
      const codedPhrase: string = user.getPhraseEncodedByPassword(); 
      const phrase: string = cryptographer.decrypt(codedPhrase, password); 
      if ("Valid Password".equals(phrase)) {
        Session.initialize();
        return true; 
      }
    }
    return false; 
  }
}
```
就行為上來說，登入後將 session 初始化似乎是不可避免的，如若這般，將 function name 改為 `checkPasswordAndInitializeSession` 會是更好的選擇。

### Output Arguments 
參數通嘗是被解讀為 function 的**輸入值**

因此與其撰寫一個包含輸出型參數的 function (where base will change)
```typescript
type append = (base: string, tail: string) => void;
```

不如單純的回傳結果
```typescript
type append = (head: string, tail: string) => string;
```

或是善用 this 的概念讓 append function 變為以下的形式都會更好
```typescript
interface String {
  append = (tail: string) => void;
}
```

## Command Query Separation
基於 function 做一件事的原則，不要讓一個 function 同時有搜尋和下指令的功能，例如以下例子
```typescript
type set = (attribute: string, value: string) => boolean;
```
這個 function 會將傳入的直寫進對應的 attribute 中，如果成功會回傳 true，反之則為 false。由於 function 同時有著指令和查詢的能力，自然而然就會產生出以下的 statement
```typescript
if (set('username', 'kevin')) {
  //...
}
```

讀者很難直接的判斷是 username 等於 kevin 時回傳 true，或是 username 設定為 kevin 後回傳 true。與其保留這種不確定性，不如將 function 設計的更單純
```typescript
if (isAttributeExists('username')) {
  setAttribute('username', 'kevin');
}
```

## Prefer Exceptions to Returning Error Codes 
有時我們會見到 function 回傳 error code 的情境，這樣的做法有點違反 Command Query Separation 的原則。
```typescript
if (deleteVariants(product.variants) === E_OK) {
  console.log('variants deleted');  
  if (deleteProduct(product) === E_OK) {
    console.log('product deleted');
  } else {
    console.error('delete product failed');  
  }
} else {
  console.error('delete variants failed');
}
```

但如果用 execption 取代 error code，不只可以遵守這個原則，也讓流程看起來更加簡潔。
```typescript
try {
  deleteVariant(product.variants); 
  deleteProduct(product);
} catch (Exception e) {
  console.error(e.getMessage()); 
}
```

- Extract Try/Catch Blocks 
- Error Handling Is One Thing
盡量將 try/catch 內的程式碼抽出為獨立的 functions，抱持 function 只做一件事(錯誤處理)的原則

- The Error.java Dependency Magnet 
回傳錯誤碼代表有列舉的狀況發生，那在新增錯誤情境時成本相對會變高。相反，如果用 try/catch 我們可以建立新的例外類別就好。

## Don’t Repeat Yourself 
**Be DRY**

## Structured Programming 
- Edsger Dijkstra’s rules of structured programming


