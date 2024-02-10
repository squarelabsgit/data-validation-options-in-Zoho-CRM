# Data Validation Options in Zoho CRM
Blog Post: https://www.squarelabs.com.au/post/data-validation-options-in-zoho-crm

YouTube Video: https://youtu.be/3xf2xX-xYpw

Ensuring data validity within a CRM system ranks among one of its most challenging tasks so in this article we will explore data validation options in Zoho CRM.

There are many things that you can do to help improve data accuracy in your CRM and one of the easiest ways to get started is to implement validation rules.

Validation rules are a set of criteria or conditions defined within the system to ensure that the data entered into a particular field meets requirements and this maintains data accuracy, consistency, and integrity. For example when a user attempts to save or update a record, the validation rule checks whether the entered data conforms to the specified conditions. If the data does not meet the criteria, the system prevents the record from being saved and typically provides an error message or notification indicating the issue.

We will explore 3 tiers of validation starting with the standard Validation Rule, followed by a Validation Rule Function, and finally the power-packed Client Script. Using the same consistent example — validating postcodes — we will illustrate how each validation type enhances and refines the validation process.

I hope that there was a few tips in this article that you didn't know about.

## Standard Validation Rules

Starting with the fundamentals, the standard Validation Rule is your go-to option for straightforward validations. It allows you to set criteria based on fields, ensuring that the data entered aligns with your predefined standards. For instance, you can create a rule that ensures a field is not left empty and that the entered data is correct based on the value of another field.

## Validation Rule Functions

Advancing up the hierarchy, Validation Rules Functions take data validation to the next level. This feature allows for more complex and dynamic validations, including the use of regular expressions. Imagine having a rule that triggers based on a combination of conditions or performs calculations to validate the entered data. This level of sophistication can be invaluable in scenarios where your validation requirements are intricate and cannot be defined by basic criteria alone.

```js
entityMap = crmAPIRequest.toMap().get("record");
response = Map();
/* ---------------------------------------------------------------------------------------------- */
postcode = entityMap.get("Billing_Code");
state = entityMap.get("Billing_State");
//Ensure that the State
if(!isnull(state))
{
	if(!isnull(postcode))
	{
		//Regex to check postcode format matches 4 digits
		if(postcode.matches("[0-9]{4}"))
		{
			//ensure that postcode is a number for the operator functions and will remove any leading 0's
			postcode = postcode.toNumber();
			//ACT
			if(state == "ACT")
			{
				if(postcode >= 200 && postcode <= 299 || postcode >= 2600 && postcode <= 2618 || postcode >= 2900 && postcode <= 2920)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//NSW
			else if(state == "NSW")
			{
				if(postcode >= 1000 && postcode <= 2599 || postcode >= 2618 && postcode <= 2899 || postcode == 3644 || postcode == 3707)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//VIC
			else if(state == "VIC")
			{
				if(postcode >= 3000 && postcode <= 3999 || postcode >= 8000 && postcode <= 8999)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//QLD
			else if(state == "QLD")
			{
				if(postcode >= 4000 && postcode <= 4999 || postcode >= 9000 && postcode <= 9999)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//SA
			else if(state == "SA")
			{
				if(postcode >= 5000 && postcode <= 5999)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//TAS
			else if(state == "TAS")
			{
				if(postcode >= 7000 && postcode <= 7999)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//WA
			else if(state == "WA")
			{
				if(postcode >= 6000 && postcode <= 6999)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			//NT
			else if(state == "NT")
			{
				if(postcode >= 800 && postcode <= 999)
				{
					response.put('status','success');
				}
				else
				{
					response.put('status','error');
					response.put('message','This postcode does not belong to this state.');
				}
			}
			else
			{
				response.put('status','error');
				response.put('message','Please enter state in the following format: NSW, VIC, SA.');
			}
		}
		else
		{
			response.put('status','error');
			response.put('message','Postcode must be a 4 digit number.');
		}
	}
	else
	{
		response.put('status','error');
		response.put('message','Please enter a postcode.');
	}
}
return response;
```

## Client Script

Finally, we arrive at the Client Script – the powerhouse of data validation in Zoho CRM. Client Scripts offer an unparalleled level of customization and control. With Client Scripts, you can execute validation logic not only during data entry but also in response to various input events, such as field changes, record loading, or record saving. Additionally, the flexibility extends to the ability to call external APIs, providing an immense range of options for robust data validation.

```js
//Get Field Objects
const billingStateField = ZDK.Page.getField('Billing_State');
const billingCodeField = ZDK.Page.getField('Billing_Code');
//Get Field Values
const billingState = billingStateField.getValue();
const billingCode = billingCodeField.getValue();
//Get API Key from CRM Variables
if (billingState && billingCode) {
    //Check the postcode is 4 numbers with a RegEx
    const postcodeRegex = /^\d{4}$/;
    if (postcodeRegex.test(billingCode))
    {
        const fetchApiKey = ZDK.Apps.CRM.Settings.fetchVariablesById('6040139000001156675');
        let apiKey = fetchApiKey.value;
        //Call Auspost API
        auspostQuery = ZDK.HTTP.request({
            url: 'https://digitalapi.auspost.com.au/postcode/search.json',
            method: 'GET',
            parameters: {
                'q': billingCode,
                'state': billingState
            },
            headers: {
                'auth-key': apiKey
            }
        });
        //Get the API Response Status
        let auspostStatus = auspostQuery.getStatusCode();;
        //Get the API Response
        let auspostResponse = auspostQuery.getResponse();
        //Only continue if a 200 status is returned
        if (auspostStatus === 200) {
            let auspostData = JSON.parse(auspostResponse);
            let suburbList = auspostData.localities.locality;
            //API returns either null/undefined || Object || Array based on number of suburbs.
            //null / undefined
            if (suburbList && suburbList !== null) {
                //If object create Array
                if (!Array.isArray(suburbList)) {
                    suburbList = [suburbList];
                }
                //Check there is at least 1 object in the Array
                if (suburbList.length > 0) {
                    //check for a match
                    const matchFound = suburbList.some(suburb => suburb.state === billingState && suburb.postcode == billingCode);
                    if (matchFound) {
                        return true;
                    }
                }
                else {
                    billingCodeField.showError('Postcode does not belong to this State.');
                    return false;
                }
            }
            else {
                billingCodeField.showError('Postcode does not belong to this State.');
                return false;
            }
        }
        else {
            let errorMessage = JSON.parse(auspostResponse).error.errorMessage
            ZDK.Client.showAlert('An Error Occured: ' + errorMessage, 'Notice', 'Got it!');
        }
    }
    else
    {
        billingCodeField.showError('Postcode must be a 4 digit number.');
        return false;
    }
}
else if (!billingState && billingCode)
{
    billingStateField.showError('Please Enter a State Value in the format: NSW, VIC, SA');
    return false;
}
else if (billingState && !billingCode)
{
    billingCodeField.showError('Please Enter a Postcode.');
    return false;
}
return true
```

## Important Things to Note

If a primary or secondary field used in a validation rule gets updated via workflow field update action, Blueprint, APIs, Import or Webforms, this field update takes precedence. Which means, the validation rule gets overwritten.

Need Help? [Contact us!](https://www.squarelabs.com.au/contact-us)

## Resources

GitHub Code: https://github.com/squarelabsgit/data-validation-options-in-Zoho-CRM

Zoho Documentation: https://help.zoho.com/portal/en/kb/crm/customize-crm-account/validation-rules

AusPost API: https://developers.auspost.com.au/apis/pacpcs-registration

<a href="http://www.youtube.com/watch?feature=player_embedded&v=3xf2xX-xYpw" target="_blank"><img src="http://img.youtube.com/vi/3xf2xX-xYpw/0.jpg" 
alt="YouTube Video" width="240" height="180" border="10" /></a>
