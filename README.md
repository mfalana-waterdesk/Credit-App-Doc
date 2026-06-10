# Credit Application - Documentation

The purpose of this document is to define the latest Water Desk Credit App Aspire API call JSON documentation - everything for the booking process. POST, PUTS, GETS, and even DELETES (if applicable), that may be a part of the process. Example scenarios can be found in Appendix A, page 8. 

## Steps

1. [Credit Application: New Customer Account(Step 1)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896860&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKz4ErBrhSiEsazNbSwNLMwM0BXnVxaXJJUAlUM5dhaGhtYGJkBAA)

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



2. [Credit Application: Create Location(New Customer) Step 2](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896861&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)

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



3. [Credit Application: Set Primary Location(New Customer) Step 3](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=1896864&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)

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

4. [Credit Application: Send Contract to Aspire(New Customer) Step 4](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=4970083&back=~v2~S0nSNzczMbHUL04tKS3QL09LTC7JzM8r1kssLqiwL0lMykm1tTQyNTKzAAA)

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
