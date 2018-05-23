# Improve & Optimize The Code
<a href="https://tareqnewazshahriar.github.io/Improve-Your-Code/">View as website</a>

#### Do not just improve or optimize or clean a code and leave. Share it, keep it... for others.

*(This is not a code repository. Here we are showing how to improve the code from real life mistakes, examples)*

#### NOTE:
----------------
* Only the code found in real life will be added which can be improved/cleaned. And this project is not for general tips, tricks.
* Sensitive, unnecessary, irrelevant code parts should be replaced/removed.
* New point will be added at the top with descend numbering.
* If a new point is needed to be added after or before another point which is at the middle, number it like ‘10.0’ or ‘10.1’ etc. But do not change any existing number.
----------------


#### SO... HERE WE GO

## [11]
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

## [10.1]
Now the reverse of below ([10]), - assign true/false in a property of an object based on a string having only characters of '0' or '1':
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

## [10]
From a list of objects add “0” or “1” in a string, based on a property of each object:
````c#
class Module
{
    public string Name { get; set; }
    public int SeqNo { get; set; }
    public bool IsChecked { get; set; }
}
````

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
string bits = string.Join("", license.Module.OrderBy(x=>x.SeqNo).Select(x => x.IsChecked ? "1" : "0")));
````

**UPDATE:** *Please remove Magic numbers with Constant/Enum/etc. with the use of intent revealing name.*

## [9]
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

## [8]
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

## [7]

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

**IMPROVE**
````c#
code.Append(license.ExpiryDate.Value.ToString("yyyyMMdd"));
````

## [6]
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

## [5]
````c#
string code = string.Empty;
code += license.Falg1 ? "0" : "1";
code += license.Falg2 ? “yyyyMMdd” : string.Empty;
code += license.Falg3 ? "0" : "1";
code += license.Falg4 ? "0" : "1";
````

**Reason:** Should use StringBuilder. For each concatenation, one new string variable will be created.
Should be used self-explanatory names which reveal it’s intent instead of Falgs and also remove Magic numbers.

## [4]
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

## [3]
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

## [2]
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

## [1]
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
````c#
if (item != null)	// Do not nest; instead skip if item is null
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
