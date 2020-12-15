# Optimize • Clean • Improve The Code
<a style="display:none" href="https://tareqnewazshahriar.github.io/Improve-Your-Code/">View as website</a>

<h3>Do not just optimize, clean or improve a code and forget. Keep it, share it,... for future.</h3>
Learn from real life mistakes and improve it, in terms of efficency, optimization and cleanliness.

<br>
<br>

**NOTE**
* Only real life code will be added which can be improved. And this project is not for general tips, tricks.
* Sensitive or irrelevant code parts should be removed/replaced.
* New point will be added at the top with descend numbering.
* If a new point is needed to add after or before another point which is at the middle, number it like ‘10.0’ (to add before 10) or ‘10.1’ (to add after 10) etc. But do not change any existing number.
* [Here](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)'s a quick Markdown guide.
<br/>
<br/>


## [16] Not using parallel execution
**Original code**
C#:
````c#
var categories = await _api.getCategories();
var boats = await _api.getBoats();
````

Problems: Here, those api methods are `async`, so we can leverage that, start them simultaneously instead of start-finish one by one.

**IMPROVED**
````c#
var getCategoriesTask = _api.getCategories();
var getBoatsTask = _api.getBoats();

var categories = await getCategoriesTask;
var boats = await getBoatsTask;
````

## [15] Redundancy
C# + HTML (cshtml) syntax:

**Original code**
````html
if (ModelList == null)
{
    <input id="port-names" type="Text" class="textcenter block bg-whitish" value="" />
}
else
{
    <input id="port-names" type="Text" class="textcenter block bg-whitish" value="@string.Join(", ", @ModelList.First().SomeString)" />
}
````
Problems:
1. If we have any change on the `class` or etc, we will have to do that on both blocks.
2. The change is only in ````value```` attribute but entire tag is duplicated.


**IMPROVED**

````html
<input id="port-names"
       type="Text"
       class="textcenter block bg-whitish"
       value="@(ModelList == null ? "" : string.Join(", ", @ModelList.First().SomeString))" />
````

Labels: #redundancy

<!-- @TareqNewazShahriar -->

## [14]
Return any teamId from the list, if the list is empty Add an item to list and return the Id.

**Original code**
````c#
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
````

**IMPROVED** <br/>
Removed redundant code.
````c#
if (list != null && list.Any())
{
    return list.First().Id;
}

var team = Team.NewTeam("Test Team", userId);
Save(team);
teamList.Add(team);
return team.TeamId;
````
<!-- @RaihanKabir -->

## [13]

List evaluation inside loop will evaluate it every time. Is it desired! Know before doing.

**Original code**
````c#
var result = listX.Where(x => ((listY.Select(y => y.Id).ToList()).Contains(x.Id))).ToList();
````

**IMPROVE** <br/>
Evaluate the intermediate list before entering into the loop, then reuse it.
````c#
var idList = listY.Select(y => y.Id);
var result = listX.Where(x => idList.Contains(x.Id)).ToList();
````
<!-- @TareqNewazShahriar -->

## [12]

Linq ForEach method is there to modify some properties of a collection. No need to create another list by copying all the properties.

**Original code**
````c#
var response = await _service.GetAll();
var list = new List<AModel>();

foreach (var item in response.Result)
{
    list.Add(new AModel
    {
        Id = item.Id,
        FromDate = DateTime.Parse(item.FromDate, new CultureInfo("en-US")).ToString("dd/MM/yyyy", CultureInfo.InvariantCulture),
        ToDate = DateTime.Parse(item.ToDate, new CultureInfo("en-US")).ToString("dd/MM/yyyy", CultureInfo.InvariantCulture),
        PolicyId = item.PolicyId,
        Count = item.Count,
        Message = item.Message,
        RequestDate = DateTime.Parse(item.RequestDate, new CultureInfo("en-US")).ToString("dd/MM/yyyy", CultureInfo.InvariantCulture),
        Boat = item.Boat,
        User = item.User,
        Phone = item.Phone
    });
}
````

**IMPROVE**
````C#
var response = await _service.GetAll();
var list = response.Result as List<Model>;

list.ForEach(x => {
    x.FromDate = DateTime.Parse(x.FromDate, new CultureInfo("en-US")).ToString("dd/MM/yyyy", CultureInfo.InvariantCulture);
    x.ToDate = DateTime.Parse(x.ToDate, new CultureInfo("en-US")).ToString("dd/MM/yyyy", CultureInfo.InvariantCulture);
    x.RequestDate = DateTime.Parse(x.RequestDate, new CultureInfo("en-US")).ToString("dd/MM/yyyy", CultureInfo.InvariantCulture);
});
````
<!-- @TareqNewazShahriar -->

## [11.1]
Now the reverse of below, - read a string and assign true/false in items of a list:

**Original code**
````c#
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
````

**IMPROVE**
````c#
license.Module = moduleList
      	            .OrderBy(x => x.SeqNo)
                    .Select((item, index) => { item.IsChecked = (moduleBits[index]=='1'); return item; })
            	    .ToList();
````
<!-- @TareqNewazShahriar -->

## [11]
Iterate through a collection of object and check for a boolean property; add “0” or “1” in a string:
````c#
class Module
{
    public string Name { get; set; }
    public int SeqNo { get; set; }
    public bool IsChecked { get; set; }
}
````

**Original code**
````c#
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
````

**IMPROVE**
````c#
string bits = string.Join("", license.Module.OrderBy(x => x.SeqNo).Select(x => x.IsChecked ? "1" : "0")));
````
<!-- @TareqNewazShahriar -->

**UPDATE:** *Please remove Magic numbers with Constant/Enum/etc. with the use of intent revealing name.* <!-- @mfhs -->

## [10]

````c#
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
````

**Original code**
````c#
if (license.LicenseType.ToString() == "Demo")
   code.Append("0");   // code is stringBuilder
else
   code.Append("1");
````

**IMPROVE**
````c#
code.Append((int)license.LicenseType);
````
<!-- @TareqNewazShahriar -->

## [9]
**Original code**
````c#
(drugclass == "C3" || drugclass == "C4" || drugclass == "C5")
````

**IMPROVE**
````c#
new string[] { "C3", "C4", "C5" }.Contains(drugclass)
````

**Reason:**
* Doesn’t need to type ‘drugclass’ every time (sometimes this may lead to an error of mistakenly typing similar property or variable).
* More items can be added (or removed) easily.
<!-- @TareqNewazShahriar -->

## [8]
**Original code**
````c#
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
````

Just found another similar line of code:
````c#
var newCodeDate = $"{DateTime.Today.Year}{DateTime.Today.Month.ToString().PadLeft(2, '0')}{DateTime.Today.Day.ToString().PadLeft(2, '0')}";
````

**IMPROVE**
````c#
code.Append(license.ExpiryDate.Value.ToString("yyyyMMdd"));
````
<!-- @TareqNewazShahriar -->

## [7]
**Original code**
````c#
if (license.NoOfUser != null)
{
    if (license.NoOfUser < 10)
    {
        code.Append("0");	  // code -> stringBuilder
        code.Append(license.NoOfUser);
    }
	else
        code.Append(license.NoOfUser);
}
else
	code.Append("00");
````

**IMPROVE**
````c#
code.Append((license.NoOfUser ?? default(int)).ToString().PadLeft(2, '0'));
````
<!-- @TareqNewazShahriar -->

## [6]
**Original code**
````c#
string code = string.Empty;
code += license.Falg1 ? "0" : "1";
code += license.Falg2 ? “yyyyMMdd” : string.Empty;
code += license.Falg3 ? "0" : "1";
code += license.Falg4 ? "0" : "1";
````

**Reason:** Should use StringBuilder. For each concatenation, one new string variable will be created.
<!-- @TareqNewazShahriar -->
**More improvement:** Should be used self-explanatory names which reveal it’s intent instead of Falgs and also remove Magic numbers. <!-- @mfhs -->

## [5]
**Original code**
````c#
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
````

**IMPROVE**
````c#
var applicationTypeList = Enum.GetValues(typeof(EnumCollection.ApplicationType))
	.Cast<EnumCollection.ApplicationType>()
	.Select(x => new SelectListItem() 
	{ 
		Text = x.ToString(),
		Value = x.ToString(),
		Selected = (license != null && license.ApplicationType == x)
	}).ToList();
````
<!-- @TareqNewazShahriar -->

## [4]
**Original code**
````c#
if (code.Substring(9, 1) == "0")
    license.ApplicationType = EnumCollection.ApplicationType.Regular;
else
    license.ApplicationType = EnumCollection.ApplicationType.Education;
````

**IMPROVE**
````c#
license.ApplicationType = (EnumCollection.ApplicationType)int.Parse(code.Substring(9, 1));
````
<!-- @TareqNewazShahriar -->

## [3]
**Original code**
````c#
string year = code.Substring(1, 4);
string month = code.Substring(5, 2);
string day = code.Substring(7, 2);
license.ExpiryDate = Convert.ToDateTime(year + "-" + month + "-" + day);
````

**IMPROVE**
````c#
license.ExpiryDate = DateTime.ParseExact(code.Substring(1, 8), "yyyyMMdd", null);
````
<!-- @TareqNewazShahriar -->

## [2]
**Original code**
````c#
if (item != null && item.fullchecked == true)
{
  ...
}
else if (item != null && item.fullchecked == false)
{
  ...
}
````

**IMPROVE**
- Do not nest the same check `item != null`, put it as parent check.
````c#
if (item != null)
{
    if (item.fullchecked)	// Not needed to check with “true” as “item.fullchecked” is itself a boolean
    {
    ...
    }
    else
    {
    ...
    }
}
````
<!-- @TareqNewazShahriar -->

## [1]
**Original code**
````c#
if (CustomerSelect == 0)
{
    IsIndividualCustomer = false;
}
else
{
    IsIndividualCustomer = true;
}
````

**IMPROVE**
````c#
IsIndividualCustomer = CustomerSelect != 0;
````

*Another version which was found too:*
````java
MenuItem item = menu.findItem(R.id.insert_menu);
if(FirebaseUtil.isAdmin)
    item.setVisible(true);
else
    item.setVisible(false);
````

*IMPROVE*
````java
MenuItem item = menu.findItem(R.id.insert_menu);
item.setVisible(FirebaseUtil.isAdmin);
````
<!-- @TareqNewazShahriar -->
