# SitecorePowerShellExamples

This includes some really simple Sitecore PowerShell samples that were used on the presentation to the Richmond Sitecore User's Group.  No warranty is expressed, implied, or thought about.  Enjoy!

## Onion
A super simple example w/ json data dumped from theonion.com on 5/1/18 that represents some content that we can import.  JSON was created with this script:
```javascript 
var articles = []; 
$("article").each(function(){ 
  articles.push({
    title: $(this).find("header h1").text(), 
    summary: $(this).find(".excerpt").text()
   })
 }); 
JSON.stringify(articles);
```
We copied the JSON string to a file, then imported into Sitecore via this script:
```PowerShell
#Load the data
$articles = Get-Content "C:/inetpub/wwwroot/sc822/website/App_Data/onion.txt" | Convertfrom-json 

#Loop the items creating new pages
foreach($article in $articles){ 
    $title = $article.title
    $summary = $article.summary
    $newPage = New-Item -Path "master:/content/Home/$($title)" -ItemType "/sitecore/templates/Sample/Sample Item"
    $newPage.Editing.BeginEdit()
    $newPage.Fields["Title"].Value = $title
    $newPage.Fields["Text"].Value = $summary
    $newPage.Editing.EndEdit()
}
#Publish all the things!
get-item . | Publish-Item
```
But to illustrate publishing items that are in workflow (Sample Item defaults to Sample Workflow) we used this script to update the workflow state:

```PowerShell
#get all items, set workflow state and publish, we could have just as easily done this above
#but for the purposes of the demo, we're working through a progression
Get-childItem -Path master:\content\home -recurse | Foreach-Object {
    $_.Editing.BeginEdit()
    $_."__Workflow state" = "{FCA998C5-0CC3-4F91-94D8-0A4E6CAECE88}"
    $_.Editing.EndEdit()
    Publish-Item $_
}
```

This script cleans up the items that were created as a part of this sample:
```PowerShell
##cleanup the onion's sample items
get-item master: -Query "fast:/sitecore/content/Home/*[@@templatename='Sample Item']" | remove-item
```

## Interactive Dialog Example
```PowerShell
#Interactive Dialogs:
if((Show-Confirm -Title "Click OK to acknowledge SPE is great!") -eq "Yes"){
    Write-host "Yeah it is!"
}
```
This example was used to show context items, and content editor button using the Module framework:
```PowerShell
if((Show-Confirm -Title "Shall I tell you the item's name?") -eq "Yes"){
    $context = get-item .
    Show-Alert -Title "Item is named... $($context.Name)" 
}
```
## Show a Report of Items currently in Workflow and show the state and step
```PowerShell
get-childitem -Path 'master:/sitecore/content/Home' -recurse | 
Where-object {$_."__Workflow" -ne $null} | 
Show-ListView -Property Name, 
    @{
        Label="Workflow"; 
        Expression={(Get-Item master: -ID $_."__Workflow").Name}
    }, 
    @{
        Label="Workflow State"; 
        Expression={(Get-Item master: -ID $_."__Workflow state").Name}
    }
```
Produces:
![Alt text](/../master/images/Report.png?raw=true "Report!")
