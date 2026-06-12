# Credit Application - Documentation

The purpose of this document is to define the latest Water Desk Credit App Aspire API call JSON documentation - everything for the booking process. POST, PUTS, GETS, and even DELETES (if applicable), that may be a part of the process. Example scenarios can be found in Appendix A, page 8. 

## Steps


### Full step-by-step walkthrough
#### New customer flow
1) Send Credit Application is clicked.
2) TeamDesk logs start time and gets dealer info.
3) **Step 1** creates customer in Aspire using [New Customer Account(Step 1)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896860) with ```POST /Account```.
4) TeamDesk moves to Step 2 to create location.
5) Step 3 assigns/sets primary location.
6) **Step 4** creates the contract in Aspire with [Send Contract to Aspire(New Customer), Step 4](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=4970083) using ```POST /Contract```, setting status to **Dealer Submitted**.
7) TeamDesk moves to Step 5 and later steps for contract info, assets, payment info, related accounts, billing-location variants, etc.
8) Status is checked later manually or by periodic triggers.
9) Aspire returns approval/decline; TeamDesk stores it in [CreditDecision](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830791) / [ContractStatus](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830792).

#### Existing customer flow
Uses [Send Contract](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=796632) instead of creating a new customer first.

---

## 1.  [Credit Application: New Customer Account(Step 1)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896860&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKz4ErBrhSiEsazNbSwNLMwM0BXnVxaXJJUAlUM5dhaGhtYGJkBAA)


****Description:**** We send the customer’s business information to Aspire so the customer exists in their system.

****Why this step exists:**** Aspire cannot process the credit application until the customer account has been created.

****What TeamDesk does:**** TeamDesk sends the company’s legal name, EIN, contact information, and customer record ID to Aspire and creates the customer as a business account.

****Result:**** A customer account is created in Aspire and is ready to be used for the rest of the credit application process.



| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Account``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json
{
"AccountType": "Business",
"Name": "<%[Company Full Legal Name]%>",
"CompanyType":{"Code":"C","Description":"Corporation"},
"Status":"Active",
"LegalName": "<%[Company Full Legal Name]%>",
"Email":"<%[New Email]%>",
"DoingBusinessAs":"<%[DBA:]%>",
"EstablishedDate": "<%ToText([Business Start Date])%>",
"FederalIdentificationNumber": "<%[EIN]%>",
"IsPrimary":true,
"PhoneNumbers": [{
"Number": "<%[AS1-Phone]%>",
"Extension":null,
"PhoneNumberType": "Main",
"IsPrimary": true
}],
"StateOfIncorporation":null,
"Contacts": [{"Value": "<%[Customer Account#]%>","Type": "Record"}],
"RecordId" : {"Type":"Record", "Value":"<%[Customer Account#]%>"},
"Roles":[{
"RoleType": "CUST",
"Description": "Customer"
}]
}
```



## 2. [Credit Application: Create Location(New Customer) Step 2](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896861&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)

****Description:**** We send the customer’s main address to Aspire so the customer has a location on file.

****Why this step exists:**** Aspire needs an address/location for the customer before the contract can be completed correctly.

****What TeamDesk does:**** TeamDesk sends the customer’s main address, city, state, country, postal code, contact name, email, phone number, and tax-exempt status to Aspire as the primary location.

****Result:**** A primary customer location is created in Aspire and linked to the customer account.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/location``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json
{
  "EntityRecordId":"<%[Customer Account#]%>",
  "Code":"Primary Location",
  "RecordId": "<%[Customer Account#]%>",
  "AccountType": "Business",
  "IsPrimary": true,
  "IsTaxExempt":"<%If([Is Tax Exempt]="Yes","True","false")%>",
  "Location":"Location",
  "Description":"<%[New City] &"Primary Location"%>",
  "Address1": "<%[New Main Address]%>",
  "Address2": "<%[New Main Address#2]%>",
  "City": "<%[New City]%>",
  "State": "<%[New State]%>",
  "Country": <%=If([Country] = "Canada", "CAN", IsNull([Country]), "USA", [Country])%>,
  "Attention": "<%[AS1-First Name]&" "&[AS1-Last Name]%>",
  "Email": "<%[AS1-Email]&"; "&[AS2-Email]%>",
  "PhoneNumbers": [{
"Number": "<%[AS1-Phone]%>",
"Extension":null,
"PhoneNumberType": "Main",
"IsPrimary": true
}],
"Name":"Primary Location",
"AddNew":true,
"IsActive":true,
"PostalCode": <%=If([Country] = "USA", [New Zip/Postal Code], [Country] = "Canada", [New Postal Code], null)%>,
"LocationRecordId": {
    "Type": "Record",
    "Value": "<%[Customer Account#]%>"
  }
}
```



## 3. [Credit Application: Set Primary Location(New Customer) Step 3](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896864&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)

**Plain English:** We tell Aspire which location should be treated as the customer’s main location.

**Why this step exists:** Even after the address is created, the customer account still needs to be pointed to that location as its primary location.

**What TeamDesk does:** TeamDesk updates the customer account in Aspire so the newly created location becomes the account’s official primary location.

**Result:** The customer account now has the correct primary location assigned in Aspire.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Account``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json
{
  "RecordId": {"Type":"Record", "Value":"<%[Customer Account#]%>"},
  "LocationRecordId": {
    "Type": "Record",
    "Value": "<%[Customer Account#]%>"
  }
}
```

## 4. [Credit Application: Send Contract to Aspire(New Customer) Step 4](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=4970083&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)

**Plain English:** We create the credit application in Aspire as a contract and submit it for review.

**Why this step exists:** This is the actual submission of the credit application to Aspire.

**What TeamDesk does:** TeamDesk sends the contract number, customer ID, bill-to location, tax location, dates, term, billing setup, and related dealer/admin information to Aspire. The contract is created with status Dealer Submitted.

**Result:** The credit application exists in Aspire as a contract and is officially submitted for processing.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Contract``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json
{
  "RecordId": {"Type": "Record","value": "<%[Contract#]%>"},
  "FinanceCompanyRecordId": {"Type": "Record", "value": <%= If([Country] = 'Canada', "10002", "10001") %> },
  "CustomerRecordId": {"Type": "Record","value": "<%If([Mavis Account Override]=True,[Mavis Account Override ID],[Account No#])%>"},
  "ContractStatus": {
      "ContractRecordId":  {"Type": "Record","value": "<%[Contract#]%>"},
      "Status": "Dealer Submitted"
    },
  "BillToLocationRecordId": {"Type": "Record","value": "<%If([Mavis Account Override]=True,[Mavis Account Override ID],[Customer Account#])%>"},
  "TaxLocationRecordId":  {"Type": "Record","value": "<%If([Mavis Account Override]=True,[Mavis Account Override ID],[Customer Account#])%>"},
    "SignDate": "<%ToText([Close Date])%>",
    "StartDate": "<%ToText([Close Date])%>",
    "Loan": null,
    "Term": "<%[Term(New)]%>",
    "TransactionCode": {
      "InvoiceDescription": "Rent",
      "Code": "RENT",
      "Description": "Rent"
    },
    "AccountDistributionCode": {
      "Code": null,
      "Description": null
    },
    "InterestCalculation": 70,
    "FinanceProgram": {
      "Code": "NON-RECOURSE",
      "Description": "Non-Recourse"
    },
    "FinanceProduct": {
      "Code": "Rental Contract",
      "Description": "Rental Contract"
    },
    "ContractType": {
      "Code": "NON RECOURSE",
      "Description": "Non Recourse"
    },
    "ContractCategory": {
      "Code": null,
      "Description": null
    },
    "DelinquencyCode": {
      "Code": "15-15",
      "Description": "15 Days 15%"
    },
    "InvoiceLeadDays": {
      "Code": "<%If([Billing Freq(New)]="Monthly","20",
                  [Billing Freq(New)]="Quarterly","45","20")%>",
      "Description":"<%If([Billing Freq(New)]="Monthly","20 Days all Monthly Contracts",
                        [Billing Freq(New)]="Quarterly","45","20")%>"
    },
    "PenaltyCode": {
      "Code": null,
      "Description": null
    },
    "InvoiceCode": {
      "Code": "BTS",
      "Description": "BillTrust-Separate"
    },
    "CompoundingPeriod": "FollowPaymentFrequency",
    "ComputeMethod": "Normal",
    "YearLength": "YearLength360",
    "SuspendInvoicing": true,
    "Eft": null,
    "RelatedAccounts": [
      {
        "ContractId": {"Type": "Record","value": "<%[Contract#]%>"},
        "EntityId": {
          "Value": "<%[Administrator Id]%>",
          "Type": "Transaction"
        },
        "Role": {
          "RoleType": "OTHER4",
          "Description": "Dealer Administrator"
        },
        "IsPrimary": true,
        "Action": "Default",
        "RecordId": null
      },
      {
        "EntityName": "<%[Dealer Name]%>",
        "ContractId": {"Type": "Record","value": "<%[Contract#]%>"},
        "EntityId": {
          "Value": "<%[Dealer No]%>",
          "Type": "Transaction"
        },
        "Role": {
          "RoleType": "VEND",
          "Description": "Dealer"
        },
        "IsPrimary": true,
        "Action": "Default",
        "RecordId": null
      }
      ]
}
```



5. [Credit Application: ]()

**Plain English:** We ask Aspire for the contract details so TeamDesk can store important information from the submitted application.

**Why this step exists:** Later steps depend on values that only Aspire can provide after the contract has been created, especially the transaction number.

**What TeamDesk does:** TeamDesk retrieves the contract record from Aspire and stores items such as contract status, credit decision, decision date, approval amount, and the Aspire transaction number (Trans#).

**Result:** TeamDesk now has the contract details it needs to continue the process and track the application.


  a) [Credit Application: Get Contract Info(New Customer Only)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1764995&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)  

  | Syntax | Description |
  | ----------- | ----------- |
  | Method | GET |
  | Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Contract/<%[Contract#]%>/record``` |
  | Headers | ```Content-Type: application/json``` |

  
    b) [Credit Application: Get Credit Status Aspire](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1765488&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA#anchor_wfsubaction)
  
  | Syntax | Description |
  | ----------- | ----------- |
  | Method | GET |
  | Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/contract/<%Trim(ToText(Format([Trans#])))%>/transaction``` |
  | Headers | ```Content-Type: application/json``` |

## 6. [Credit Application: ]()

**Plain English:** We send the main piece of equipment or unit to Aspire and attach it to the contract.

**Why this step exists:** Aspire needs to know what asset or equipment is part of the financing agreement.

**What TeamDesk does:** TeamDesk sends the asset description, asset code, serial number, quantity, funding amount, location, vendor/distributor, and contract transaction number to Aspire.

**Result:** The primary asset is created in Aspire and associated with the submitted contract.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ``` ``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json

```


## 7. [Credit Application: ]()

**Plain English:** We connect the asset more fully to the contract record inside Aspire.

**Why this step exists:** After the asset is created, Aspire may need an additional linking step so the asset is properly tied to the contract structure.

**What TeamDesk does:** TeamDesk runs the next contract/asset linking step so Aspire treats the asset as part of the contract’s financed items.

**Result:** The asset is tied into the contract record and is ready for payment and location updates.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ``` ``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json

```

## 8. [Credit Application: Send the Payment Information(Primary)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1897499)

**Plain English:** We tell Aspire how the customer will pay for the contract.

**Why this step exists:** The contract is not financially complete until the payment schedule and amounts are defined.

**What TeamDesk does:** TeamDesk sends the term, contract start date, funding amount, billing frequency, number of payments, and calculated payment amount to Aspire. It also adjusts the payment start date and amount depending on whether billing is monthly, quarterly, semiannual, or annual.

**Result:** The contract in Aspire now has its payment schedule and financial terms set up.

| Syntax | Description |
| ----------- | ----------- |
| Method | PUT |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Contract``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json
{
"FinanceCompanyRecordId": {"Type": "Record", "value": <%= If([Country] = 'Canada', 10002, 10001) %> },
"CustomerRecordID": {"value": "<%If([Mavis Account Override]=True,[Mavis Account Override ID],[Customer Account#])%>","type": "Record"},
"RecordId": {"value": "<%[Contract#]%>","type":"Record"},
"ContractStatus": {"Status":"Dealer Submitted"},
"Term": "<%[Term(New)]%>",
"StartDate": "<%ToText([Close Date])%>",
"OriginalCost": "<%ToText([Potential Funding])%>",
"Lease": {
"Payments": [
  {
  "StartDate": "<%If([Billing Freq(New)]="Quarterly",ToText(AdjustMonth([Close Date],3)), [Billing Freq(New)]="SemiAnnual",ToText(AdjustMonth([Close Date],6)),
                    [Billing Freq(New)]="Annual",ToText(AdjustMonth([Close Date],12)),ToText(AdjustMonth([Close Date],1)))%>",
  "Occurrences": "<%Left(ToText([No of Payments]),".")%>",
  "Frequency": "<%[Billing Freq(New)]%>",
  "Amount": "<%If([Billing Freq(New)]="Quarterly",ToText([Total_Monthly_Payment_Amount]*3), [Billing Freq(New)]="SemiAnnual",ToText([Total_Monthly_Payment_Amount]*6),
                  [Billing Freq(New)]="Annual",ToText([Total_Monthly_Payment_Amount]*12),ToText([Total_Monthly_Payment_Amount]))%>",
  "PaymentType": "Standard"
      }

    ]
  }
}
```



## 9. [Credit Application: Move Asset to Contract(Primary)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1888760)

**Plain English:** We connect the asset to the correct contract location and billing location in Aspire.

**Why this step exists:** The asset needs to point to the right customer/service location and, if applicable, the right bill-to location.

**What TeamDesk does:** TeamDesk updates the asset inside Aspire so it uses the correct customer location and bill-to location for the contract.

**Result:** The asset is fully connected to the contract’s final location and billing structure in Aspire.

| Syntax | Description |
| ----------- | ----------- |
| Method | PUT |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/contract/<%[CA-ContractID]%>/record/asset``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json
[
{
"IsPrimary":true,
"RecordId": {
        "Value": "<%[Asset ID]%>",
        "Type": "Record"
      },
"LocationId":"<%If([Mavis Account Override]=True,[Mavis Account Override ID],[Customer Account#])%>",
"BillToLocationId":"<%If([Use Customer Address]=false,[BilltoLoc],[Mavis Account Override]=True,[Mavis Account Override ID],[Customer Account#])%>"
}
]
```



#. [Credit Application: ]()

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ``` ``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |

```json

```

## Record Change Triggers

## Attaching Units
