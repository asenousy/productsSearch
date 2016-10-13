# Description

This Web application uses a JSON file as a data source to display of all the product titles and gives the ability to search for a product based on product title,
the application will also allow you to refine your search result with more keywords entered


## Screenshot
![Alt text](https://github.com/asenousy/productsSearch/blob/master/ScreenShot.png)


## Strategies

1. load JSON
2. search the Json product Titles for keywords requested
3. refine the search result
4. fill table with result

### JSON File

the products are listed in JSON by worksById as you see below where each ID contains many product details, our focus will be to the "TitleText" which is inside "Title" key

```
"worksById": {
    "9780000001306": {
        "NotificationType": "03",
        "ProductIdentifier": [
            {
                "ProductIDType": "03",
                "IDValue": "9780000001306"
            },
            {
                "ProductIDType": "15",
                "IDValue": "9780000001306"
            }
        ],
        "Barcode": "03",
        "ProductForm": "BC",
        "Title": {
            "TitleType": "01",
            "TitleText": "CAM Testing: BJ Migration patch 1 2011..."
			},
			
			...
			...

			
	"9780000001962": {
        "NotificationType": "03",
        "ProductIdentifier": [
            {
                "ProductIDType": "03",
                "IDValue": "9780000001962"
            },
            {
                "ProductIDType": "15",
                "IDValue": "9780000001962"
            }
        ],
        "Barcode": "03",
        "ProductForm": "BC",
        "Title": {
            "TitleType": "01",
            "TitleText": "migration testing EW interface 10.5 - NOV sara",
            "Subtitle": "Nov sara"
			},
			
			...
			...
```


## Load JSON file

we use XMLHttpRequest and as soom as its ready the responseText property returns the JSON as a text string which we can convert to a Json object using JSON.parse

```
// load the json file
var xmlhttp = new XMLHttpRequest();
xmlhttp.onreadystatechange = function() {
	if (this.readyState == 4) {
		var objects = JSON.parse(this.responseText);
		myFunction(objects);
	}
};
var url = "ELSIO-Graph.json";
xmlhttp.open("GET", url, true);
xmlhttp.send();
```


## Searching

we need to split the input string into an array of seperate substrings to search for each keyword seperately

```
var input = document.getElementById("textbox").value;
var inputArray = input.split(" ");
```

first we extract all the IDs in the JSON into an array as below

```
var productsID =  Object.keys(obj["worksById"]); // list all keys (IDs)
```

then we end up with two arrays, an ID list and a keywords list, so using 2 for loops, we iterate and compare till we find a match

```
function searchTitle(obj, searchArray)
{
	var productAray = [];
	var productsID =  Object.keys(obj["worksById"]); // list all keys (IDs)
		
	// loop through each workID 
	for (var i = 0; i < productsID.length; i++) {
		var ID = obj["worksById"][productsID[i]];
		var Title = ID["Title"];
		var TitleText = Title["TitleText"];
			
		// loop through each keyword to check if it exists in Title
		for (var j = 0; j < searchArray.length; j++) {
				
			// check for matches
			if (TitleText.toUpperCase().includes(searchArray[j].toUpperCase())) {
					
				// first check if it already exists in our result list, if not then add it, but of not then just increment the Hits tag
				if (!isDuplicate(productAray, productsID[i])) {	
					productAray.push({"ID":productsID[i], "Title":TitleText.toUpperCase(), "Hits":1});
			} else {		
					productAray[productAray.length-1]["Hits"]+= 1;
				}
			}
		}
	}
	return productAray;
};
```

once a match is found we add the product details into a new JSON array created, ignore the Hits tag for now will explain later


## Refine search result

when searching for a product you would expect the top of the result to list the title which contains the most keywords you entered and and the product with the least keywords at the bottom, to do this I added a key called Hits, which records the amount of matches a particular product contains as seen below

```
result will look like this
= [{"ID":"9780000001306" , "Title":"CAM Testing: BJ Migration patch 1 2011" , "Hits":"2"},
	{"ID":"9780000001962" , "Title":"migration testing EW interface 10.5 - NOV sara" , "Hits":"1"},
	{"ID":"9780012752159" , "Title":"SEMICONDUCTORS & SEMIMETALS V21D" , "Hits":"3"},
	{"ID":"9780030031298" , "Title":"Fundamentals of Chemistry" , "Hits":"1"}]
```

so if a match took place then the product details is inserted into our list, but if its already there then increment the Hits tag to records how many keywords it contains

```
if (!isDuplicate(productAray, productsID[i])) {					
	productAray.push({"ID":productsID[i], "Title":TitleText.toUpperCase(), "Hits":1});
} else {
	productAray[productAray.length-1]["Hits"]+= 1;
}
```


## Fill the Table

once we have the result list, we need to sort it using the Hits value, so that the product with the highest Hits is listed on top and the product with the lowest Hits is listed in bottom


```
resultArray.sort(function(a, b){return b["Hits"] - a["Hits"]});
```

Finally we fill the table with the new sorted list

```
var tableData="";
for (var i = 0; i < resultArray.length; i++) {
	tableData +='<tr><td>'+resultArray[i]["ID"]+'</td><td>'+resultArray[i]["Title"]+'</td></tr>';
}
document.getElementById("tableBody").innerHTML=tableData;
```



