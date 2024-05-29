# assignment-zksystems

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn run serve
```

### Compiles and minifies for production
```
yarn run build
```

### Lints and fixes files
```
yarn run lint
```



# Coding assignment details

User Interface Specification
Note: The numbers are not part of the user interface. They are just for identifying the backend integration. 
The data that should be monitored can be found here. 



CSV File Explanation

Example line: 2019-04-01 0:02:00,"0,05","86,62","0,00","49923,70"
2019-04-01 0:02:00: Timestamp
"0,05": MW
"86,62" IGV Position
"0,00" Flame on
"49923,70" EOH (can be ignored)
TurnUp Dashboard

The home screen should monitor the usage of the machine SGT5-4000F as well as the amount that needs to be paid by the customer based on the usage. This machine generates electricity.
The amount will be compared with the stock market price, which can be found here. 

TurnUp detection and categories 
A TurnUp will be detected, when the following conditions are met
Flame On = 1
IGV Position: 
Minimum: 100.5 <= IGV <= 103.5
Medium: 103.5 < IGV <= 107.3
Maximum: Max: 107.3 < IGV



Pricing formula
A minute during the turnup during the categories are priced as follows.
Category
Price per minute in €
Minimum
0,42
Medium
1,25
Maximum
2,08


Simulation of streaming data 
The input signal data should be simulated as it would be streamed. That means based on the date and timestamps in the first column the data will be monitored on the user interface according to this time.
Colours: 
#50BED7 
#3296B9
#095F87
#309999

Widget 1
This widget should show the sum of all TurnUp MW starting from the first day of the current month. 

Widget 2
This widget should show the sum of the price for the current month beginning from the first day of the current month.

Widget 3 
The currently simulated date and timestamp should be displayed. This gif should be integrated as demonstration of the machine. As well as displaying the line with “Online, 2 units running”. 

Widget 4 
The graph should display the amount of usage of the TurnUps in different categories beginning from the first of the simulated month. 

Widget 5 
The light blue graph should display in real time the price that needs to be paid. The dark blue graph is monitoring the current stock market price of the electricity for that hour this data can be found in this file. The purpose is to monitor the delta between the paid price and the current price on the stock market. 

Please implement the same navigation bar for all other remaining screens.

Invoice


Note: The presentation of the date should be formatted in MM.DD.YYYY



The invoices for every month in the folder invoices should be displayed in the browser. After clicking on “DOWNLOAD INVOICE” the invoice should be downloaded.



Transactions 



The transactions displayed here (you can copy it from the invoices) can be hardcoded and displayed in a table. 
The table should display every Turnup in detail. Including the Date, Category, Duration and Price for that TurnUp. The validated is static and does not depend on a parameter. 
Note: The Tablehead is not aligned. Please implement it better ;)


Smart Contract



The left text box displays the payment formula. After clicking the “SAVE” a notification should come up that the new payment formula has been saved.
The right hand side contains an editor with the smart contract. You can edit any kind of code, it does not need to be compliant. 


Deployment

Please deploy it on a URL you can share. 

Documentation

Please provide proper documentation and comments in the code. 

