# Real-life Coding Mistakes
<!-- <a style="display:none" href="https://tareqnewazshahriar.github.io/real-life-coding-mistakes/">View as website</a> -->

**Our learning from many years of mistakes, will be a short and sweet path for others.**  
**Do not just fix or improve a code and forget; keep it, share it - for the next generation.**

This is a end-less project to keep history of coding mistakes that are found in real-life projects. Anyone can add code here. In fact everyone is highly encouraged to create pull requests to add found mistakes. This is really usefull for programmers of every stage.
<br>
Finally... isn't it appealing to look back and see the memory of coding mistakes! So please... go ahead and create a pull request.

<br>
<br>

## Important NOTE
* Replace or remove sensitive/irrelevant code parts.
* New point will be added at the top with descend numbering.
* If a new relevant point is needed to be added after or before another point which is at the middle, number it like ‘10.0’ (to add under 10) or ‘10.1’ (to add above 10). But do not change any existing number.
* Only real-life mistakes will be added. For coding tips, tricks, optimization, contribute to that project https://github.com/TareqNewazShahriar/coding-tips-tricks-optimization.
<br/>

<a href="https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet">Here<a/>'s a quick Markdown guide, who needs it.
<br/>
<br/>


## [25] Framing a filter logic in a way so that it becomes least readable and non-optimizable by the Query Engine:

**Code found in real project** (SQL) 
```sql
WHERE CategoryName = CASE
                       WHEN ISNULL(@category_name, '') = '' THEN CategoryName
                       ELSE @category_name
                     END
```

**IMPROVED**  
 Use of function with an argument in Where clause make the query non-optimizable by the engine.  
 Also the query is less readable.

```sql
WHERE @category_name is null OR @category_name = '' OR CategoryName = @category_name
```
How to frame queries so that Query Engine can optimize (sargable (Search ARGument ABLE)) it:
https://stackoverflow.com/questions/799584/what-makes-a-sql-statement-sargable



## [24] Unnecessarily complicated WHERE clause and wrong use of CASE - SQL
tags: complicated

**Code found in real project** (SQL) 
```sql
SELECT *
FROM Funds
WHERE AllocationId = CASE
                        WHEN @AllocationId IS NULL THEN AllocationId
                        ELSE @AllocationId
                     END
```

**IMPROVED**  
```sql
SELECT *
FROM Funds
WHERE @AllocationId IS NULL OR AllocationId = @AllocationId
```
  

## [23] Doing operations inside loop which is not related to loop moreover costly (watch out what you are doing inside loop):
  
  Here redundant database calls are making inside loop over and over.  
  
   **Code found in real project** (C# & LINQ)
  ```c#
   foreach (var newFund in newFunds) // New funds gathered from an excel file
   {
      var existingRecord = _accountRepository.GetExistingFundsByAccountId(accountId)).Where(x => x.UniqueCode == newFund.UniqueCode); // Where() returns a collection
      if (existingRecord == null || existingRecord.Count() == 0)
      {
         // Add new record
         await _accountRepository.AddAccountFund(newFund);
      }
      else
      {
         // Update existing record based on UniqueCode
         newFund.Id = existingRecord.First().Id;
         await _accountRepository.UpdateAccountFund(newFund);
      }
   }
   ```
  
   **IMPROVED**  
  
   * Prepare your stuffs before entering into the loop.
  
   ```c#
   // Get the existing funds before entering into the loop:
   var existingFundsInAccount = _accountRepository.GetExistingFundsByAccountId(accountId);
   
   foreach (var newFund in newFunds)
   {
      var existingFund = existingFundsInAccount.SingleOrDefault(x => x.UniqueCode == newFund.UniqueCode);
      if ()
      {
         // Update existing record based on UniqueCode
         newFund.Id = existingFund.Id;
         await _accountRepository.UpdateAccountFund(newFund);
      }
      else
      {
         // Add new record
         await _accountRepository.AddAccountFund(newFund);
      }
   }
   ```
  
 ### Here's another one of it (code unrelated with loop):
  
  Everytime it finds a file, it is creating and opening a new FTP session.
  
  **Code found in real project** (C#)
  ```c#
  string folderPath = "<local folder path>";
  string destinationFtpUrl = "<ftp url>";

  foreach (string filePath in Directory.EnumerateFiles(folderPath, "*.xlsx"))
  {
      using (FileStream fs = File.OpenRead(filePath))
      {
          Session session = new Session();  // WinSCP session
          session.Open(sessionOptionsObj);

          // Uploading file on SFTP  
          session.PutFile(fs, destinationFtpUrl, new TransferOptions { TransferMode = TransferMode.Binary });
      }
  }
  ```
  
  **IMPROVED**
  
  - Create and open a single FTP session before entering into the loop then transfer all the files with that session.
  
  ```c#
  string folderPath = "<local folder path>";
  string destinationFtpUrl = "<ftp url>";

  Session session = new Session();  // WinSCP session
  session.Open(sessionOptionsObj);

  foreach (string filePath in Directory.EnumerateFiles(folderPath, "*.xlsx"))
  {
      using (FileStream fs = File.OpenRead(filePath))
      {
          // Uploading file on SFTP  
          session.PutFile(fs, destinationFtpUrl, new TransferOptions { TransferMode = TransferMode.Binary });
      }
  }
  ```
  
   
## [22] Doing unnecessary operation just to keep the code in one line:
   
   **Code found in real project** (JS)
   ```js
   price = price ? price : 0;
   ```
   
   **IMPROVED**
   We don't need to do `price = price` when `price` has value. So do what is only need to do; even it needs two lines.
   ```js
   if(!price)
      price = 0;
   ```
   
   
## [21] Lengthy:
   
   
   **Code found in real project** (JS)
   ```js
   let price = document.querySelector('.price').value;
   price = price ? price : 0;
   let result = price * 10;
   ```
   
   **IMPROVED**
   ```js
   let price = document.querySelector('.price').value;
   let result = (price || 0) * 10;
   ```
   
   
## [20] Confusing and probably a mistake
   
   **Code found in real project** (C# & LINQ)
   ```c#
   if (products.Any())
      return products;
   return null;
   ```
   
   **IMPROVED**
   
   What is intended here?  
   
   * If inteded code is - if product empty (this check implies that `products` will never be null, since there's no null check) then return null, otherwise return the products -- then this code is fine.
   
   * If the `products` is null return null otherwise return the list? Then this check is meaningless. Simply write:
   ```diff
-   if (products.Any())
-      return products;
-   return null;
+   return products;
   ```
   
   * If desired code is - if the `products` is null or empty then return null, otherwise return the list, then the code can be:
   ```c#
   return products is object && products.Any() ? products : null;
   // OR
   return products?.Any() == true ? products : null;
   ```
   

## [19] Writing verbose or lengthy code without using null-conditional (?./?[]) and null-coalcasing operators (??/??=):
   
**Code found in real project** (C#)
```c#
if(obj.Name != null)
   obj.OtherName = obj.Name.ToUpper();
else
   obj.OtherName = null;
```

**IMPROVED**
```c#
obj.OtherName = obj.Name?.ToUpper();
```

If we need to assign some values when null, then:
```c#
obj.OtherName = obj.Name?.ToUpper() ?? "wow";
```

Note: Null-conditional operator introduced in C# 6.0 on 2015 and null-coalcasing operators introduced in C# 8 on 2019.


## [18] It's just wrong
*Labels: mistake*

**Code found in real project** (C#)
```c#
public string GetCurrency(string countryCode)
{
   return countryCode == "US" ? "USD" : AppConstants.DefaultCurrency; // currently, AppConstants.DefaultCurrency = "EUR"
}
```


**Warning**: Can you imagine what will happen on a later point, if default currency decided to be USD!


## [17] Unnecessarily lengthy code (C# specific)
*Labels: lenghty*
    
What was mainly tried to do:
- we have a currency-code (EUR/USD/...) in string; and a list of CultureInfo object.
- we need the cultureInfo object which has that currency code.


**Code found in real project** (C#)
```c#
var currencyCulture = CultureInfo.CurrentCulture;
var cultures = localizationOptions.SupportedCultures.Select(x => x.Name).ToList();
foreach (var culture in cultures)
{
   var ri = new RegionInfo(culture);
   if (ri.ISOCurrencySymbol == currencyCode) // currencyCode is a string variable
   {
      currencyCulture = CultureInfo.CreateSpecificCulture(culture);
      break;
   }
}
```

**IMPROVED**
```c#
var currrencyCulture = localizationOptions.SupportedCultures.SingleOrDefault(x => new RegionInfo(x.Name).ISOCurrencySymbol == currencyCode);
```


## [16] Not leveraging parallel execution
**Code found in real project** (C#)
```c#
var categories = await _api.getCategories();
var boats = await _api.getBoats();
```

**IMPROVED**
```c#
var getCategoriesTask = _api.getCategories();
var getBoatsTask = _api.getBoats();
// So both methods/http-requests will be executed simultaneously (depends on machine cores)

var categories = await getCategoriesTask;
var boats = await getBoatsTask;
```

## [15] Duplication of code
*Labels: duplication*
    
C# + HTML (cshtml) syntax:

**Code found in real project**
```html
if (ModelList == null)
{
    <input id="port-names" type="Text" class="textcenter block bg-whitish" value="" />
}
else
{
    <input id="port-names" type="Text" class="textcenter block bg-whitish" value="@string.Join(", ", @ModelList.First().SomeString)" />
}
```

The change is only in ```value``` attribute but the entire tag is duplicated. So the problem is-

* If we have any change on anywhere other than value attribute, we will have to keep it in mind that we will have to do that on both blocks.


**IMPROVED**

```html
<input id="port-names"
       type="Text"
       class="textcenter block bg-whitish"
       value="@(ModelList == null ? "" : string.Join(", ", @ModelList.First().SomeString))" />
```

Labels: #redundancy

<!-- @TareqNewazShahriar -->

## [14]

**What this code is trying to do**
Returns first TeamId from a team list; if the list is empty add an item to the list and return the Id.

**Code found in real project**
```c#
if (teamList != null)
{
   if (teamList.Count > 0)
      return teamList.FirstOrDefault().Id;
   else
   {
      var team = Team.NewTeam("Test Team", userId);
      Save(team);
      teamList.Add(team);
      return team.TeamId;
   }
}

var team = Team.NewTeam("Test Team", userId);
Save(team);
teamList.Add(team);
return team.TeamId;
```


**IMPROVED** <br/>
Removed redundant code.
```c#
if (list != null && list.Any())
{
    return list.First().Id;
}

var team = Team.NewTeam("Test Team", userId);
Save(team);
teamList.Add(team);
return team.TeamId;
```
<!-- @RaihanKabir -->

## [13]

List evaluation inside loop will evaluate it every time. Is it desired! Know before doing.

**Code found in real project**
```c#
var result = listX.Where(x => ((listY.Select(y => y.Id).ToList()).Contains(x.Id))).ToList();
```

**IMPROVED** <br/>
Evaluate the intermediate list before entering into the loop, then reuse it.
```c#
var idList = listY.Select(y => y.Id);
var result = listX.Where(x => idList.Contains(x.Id)).ToList();
```
<!-- @TareqNewazShahriar -->

## [12]

Linq ForEach method can be used to modify some properties of a collection. No need to create an intermediate list.

**Code found in real project**
```c#
var response = await _service.GetAll();
var list = new List<AModel>();

foreach (var item in response.Result)
{
   list.Add(new AModel
   {
      Count = item.Count + 1
   });
}
```

**IMPROVED**
```C#
var response = await _service.GetAll();
var list = response.Result as List<Model>;

list.ForEach(x => {
   x.Count = x.Count + 1  // Or just x.Count++
});
```
<!-- @TareqNewazShahriar -->

## [11.1]
Now the reverse of below, - read a string and assign true/false in items of a list:

**Code found in real project**
```c#
string module = code.Substring(13, moduleList.Count);
string[] moduleArr = module.ToCharArray().Select(c => c.ToString()).ToArray();
List<Module> objModuleList = new List<Module>();
 
int count = 0;
for (int i = 0; i < moduleArr.Length; i++)
{
    foreach (var objModule in moduleList)
    {
        if (objModule.SeqNo == count)
        {
           if (Convert.ToInt16(moduleArr[i]) == 1)
           {
                objModule.IsChecked = true;
               objModuleList.Add(objModule);
               break;
           }
        }
    }
    count++;
}
```

**IMPROVED**
```c#
license.Module = moduleList
                     .OrderBy(x => x.SeqNo)
                    .Select((item, index) => { item.IsChecked = (moduleBits[index]=='1'); return item; })
                   .ToList();
```
<!-- @TareqNewazShahriar -->

## [11]
Iterate through a collection of object and check for a boolean property; add “0” or “1” in a string:
```c#
class Module
{
    public string Name { get; set; }
    public int SeqNo { get; set; }
    public bool IsChecked { get; set; }
}
```

**Code found in real project**
```c#
int count = 0;
foreach (var module in license.Modules)
{
    code = ModuleCode(code, license.Modules, count);   // code -- stringBuilder
    count++;
}

private StringBuilder ModuleCode(StringBuilder code, List<Module> modulelist, int seq)
{
    foreach (var module in modulelist)
    {
        if (module.SeqNo == seq)
        {
           if (module.IsChecked)
           {
               code.Append("1");
               return code;
           }
        }
    }
 
    return code.Append("0");
}
```

**IMPROVED**
```c#
string bits = string.Join("", license.Module.OrderBy(x => x.SeqNo).Select(x => x.IsChecked ? "1" : "0")));
```
<!-- @TareqNewazShahriar -->

**UPDATE:** *Please remove Magic numbers with Constant/Enum/etc. with the use of intent revealing name.* <!-- @mfhs -->

## [10]

```c#
public enum LicenseType
{
   Demo,
   Licensed
}
public class License
{
   ...
   public LicenseType LicenseType { get; set; }
   ...
}
```

**Code found in real project**
```c#
if (license.LicenseType.ToString() == "Demo")
   code.Append("0");   // code is stringBuilder
else
   code.Append("1");
```

**IMPROVED**
```c#
code.Append((int)license.LicenseType);
```
<!-- @TareqNewazShahriar -->

## [9]
**Code found in real project**
```c#
(drugclass == "C3" || drugclass == "C4" || drugclass == "C5")
```

**IMPROVED**
```c#
new string[] { "C3", "C4", "C5" }.Contains(drugclass)
```

**Reason:**
* Doesn’t need to type ‘drugclass’ every time (sometimes this may lead to an error of mistakenly typing similar property or variable).
* More items can be added (or removed) easily.
<!-- @TareqNewazShahriar -->

## [8]
**Code found in real project**
```c#
// code -- stringBuilder
if (license.ExpiryDate != null)
{
    code.Append(license.ExpiryDate.Value.Year.ToString());
    if (license.ExpiryDate.Value.Month< 10)
    {
        code.Append("0");
        code.Append(license.ExpiryDate.Value.Month.ToString());
    }
    else
    {
        code.Append(license.ExpiryDate.Value.Month.ToString());
    }
 
    if (license.ExpiryDate.Value.Day< 10)
    {
        code.Append("0");
        code.Append(license.ExpiryDate.Value.Day.ToString());
    }
    else
    {
        code.Append(license.ExpiryDate.Value.Day.ToString());
    }
}
```

Just found another similar line of code:
```c#
var newCodeDate = $"{DateTime.Today.Year}{DateTime.Today.Month.ToString().PadLeft(2, '0')}{DateTime.Today.Day.ToString().PadLeft(2, '0')}";
```

**IMPROVED**
```c#
code.Append(license.ExpiryDate.Value.ToString("yyyyMMdd"));
```
<!-- @TareqNewazShahriar -->

## [7]
**Code found in real project**
```c#
if (license.NoOfUser != null)
{
    if (license.NoOfUser < 10)
    {
        code.Append("0");     // code -> stringBuilder
        code.Append(license.NoOfUser);
    }
   else
        code.Append(license.NoOfUser);
}
else
   code.Append("00");
```

**IMPROVED**
```c#
code.Append((license.NoOfUser ?? default(int)).ToString().PadLeft(2, '0'));
```
<!-- @TareqNewazShahriar -->

## [6]
**Code found in real project**
```c#
string code = string.Empty;
code += license.Falg1 ? "0" : "1";
code += license.Falg2 ? “yyyyMMdd” : string.Empty;
code += license.Falg3 ? "0" : "1";
code += license.Falg4 ? "0" : "1";
```

**Reason:** Should use StringBuilder. For each concatenation, one new string variable will be created.
<!-- @TareqNewazShahriar -->
**More improvement:** Should be used self-explanatory names which reveal it’s intent instead of Falgs and also remove Magic numbers. <!-- @mfhs -->

## [5]
**Code found in real project**
```c#
var applicationTypeList = new List<SelectListItem>();
foreach (EnumCollection.ApplicationType applicationType in Enum.GetValues(typeof(EnumCollection.ApplicationType)))
{
    if (license != null && (license.ApplicationType == applicationType))
    {
        applicationTypeList.Add(new SelectListItem 
            {
                Text = Enum.GetName(typeof(EnumCollection.ApplicationType), applicationType),
                Value = applicationType.ToString(),
                Selected = true
            });
    }
    else
   {
        applicationTypeList.Add(new SelectListItem
            {
                Text = Enum.GetName(typeof(EnumCollection.ApplicationType), applicationType),
                Value = applicationType.ToString()
            });
   }
}
```

**IMPROVED**
```c#
var applicationTypeList = Enum.GetValues(typeof(EnumCollection.ApplicationType))
   .Cast<EnumCollection.ApplicationType>()
   .Select(x => new SelectListItem() 
   { 
      Text = x.ToString(),
      Value = x.ToString(),
      Selected = (license != null && license.ApplicationType == x)
   }).ToList();
```
<!-- @TareqNewazShahriar -->

## [4] Enum
    
**Code found in real project**
```c#
// The Enum
// enum ApplicationType { Regular 0, ApplicationType.Education = 1 }

if (code.Substring(9, 1) == "0")
    license.ApplicationType = ApplicationType.Regular;
else
    license.ApplicationType = ApplicationType.Education;
```

**IMPROVED**
```c#
license.ApplicationType = (EnumCollection.ApplicationType)int.Parse(code.Substring(9, 1));
```
    
<!-- @TareqNewazShahriar -->
    
    

## [3]
**Code found in real project**
```c#
string year = code.Substring(1, 4);
string month = code.Substring(5, 2);
string day = code.Substring(7, 2);
license.ExpiryDate = Convert.ToDateTime(year + "-" + month + "-" + day);
```

**IMPROVED**
```c#
license.ExpiryDate = DateTime.ParseExact(code.Substring(1, 8), "yyyyMMdd", null);
```
<!-- @TareqNewazShahriar -->

## [2]
**Code found in real project**
```c#
if (item != null && item.fullchecked == true)
{
  ...
}
else if (item != null && item.fullchecked == false)
{
  ...
}
```

**IMPROVED**
- Do not add the same check on every check of the ladder; put it as parent check, i.e. make nested checks. 
* If the common check is needed to be changed, you have to keep all checks in mind (or you have to review every lines) then you will have to chnage every line.
* Also, if you are typing those checks in the first place, you may do a typing mistake and you will lose some time to debug it.
```c#
if (item != null)
{
    if (item.fullchecked)   // Not needed to check with “true” as “item.fullchecked” is itself a boolean
    {
    ...
    }
    else
    {
    ...
    }
}
```
<!-- @TareqNewazShahriar -->

## [1]
**Code found in real project**
```c#
if (CustomerSelect == 0)
{
    IsIndividualCustomer = false;
}
else
{
    IsIndividualCustomer = true;
}
```

**IMPROVED**
```c#
IsIndividualCustomer = CustomerSelect != 0;
```

*Another version which was found too:*
```java
if(isAdmin)
    menuItem.setVisible(true);
else
    menuItem.setVisible(false);
```

*IMPROVE*
```java
menuItem.setVisible(isAdmin);
```
<!-- @TareqNewazShahriar -->
