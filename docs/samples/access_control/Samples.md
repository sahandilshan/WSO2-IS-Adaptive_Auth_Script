## Control access based on user group
Allows login to the users who belong to any of the given set of groups.

### Prerequisites
Change the groupsToAllowAccessparameter to an array of groups for which users can access the application.

### Parameters
`groupsToAllowAccess`: An array of user groups that can access the application.

```javascript
// This script will allow access for any user who belongs
// to one of the given groups.
// If the user is a member of the following groups, user will be given access.
var groupsToAllowAccess = ['manager','employee'];

// Error page to redirect unauthorized users,
// can be either an absolute url or relative url to server root, or empty/null
// null/empty value will redirect to the default error page
var errorPage = '';

// Additional query params to be added to the above url.
// Hint: Use i18n keys for error messages
var errorPageParameters = {
    'status': 'Unauthorized',
    'statusMsg': 'You are not authorized to login to this application.'
};

var onLoginRequest = function(context) {
    executeStep(1, {
        onSuccess: function (context) {
            // Extracting authenticated subject from the first step.
            var user = context.currentKnownSubject;
            // Checking if the user is assigned to one of the given groups.
            var isMember = isMemberOfAnyOfGroups(user, groupsToAllowAccess);
            if (!isMember) {
                sendError(errorPage, errorPageParameters);
            }
        }
    });
};
```

## Control access based on user age
Allows to log in to application if the user's age is over the configured value. User's age is calculated using the user's date of birth attribute.

### Prerequisites
* Change the parameters at the top of the script as needed to match the requirements. 
* Modify the authentication option(s) from defaults as required. 
* Birth Date of the user trying to login needs to be updated using the WSO2-IS myaccount portal.

### Parameters
`ageLimit`:	Minimum age required for the user to log in to the application
`errorPage`: Error page to redirect the user if the age limit is below ageLimit
`errorPageParameters`: Parameters to be passed to the error page

```javascript
// This script will only allow login to application if the user's age is over configured value
// The user will be redirected to an error page if the date of birth is not present or user is below configured value

var ageLimit = 18;

// Error page to redirect unauthorized users,
// can be either an absolute url or relative url to server root, or empty/null
// null/empty value will redirect to the default error page
var errorPage = '';

// Additional query params to be added to the above url.
// Hint: Use i18n keys for error messages
var errorPageParameters = {
    'status': 'Unauthorized',
    'statusMsg': 'You need to be over ' + ageLimit + ' years to login to this application.'
};

// Date of birth attribute at the client side
var dateOfBirthClaim = 'http://wso2.org/claims/dob';

// The validator function for DOB. Default validation check if the DOB is in YYYY-MM-dd format
var validateDOB = function (dob) {
    return dob.match(/^(\d{4})-(\d{2})-(\d{2})$/);
};

var onLoginRequest = function(context) {
    executeStep(1, {
        onSuccess: function (context) {
            var underAge = true;
            // Extracting user store domain of authenticated subject from the first step
            var dob = context.currentKnownSubject.localClaims[dateOfBirthClaim];
            Log.debug('DOB of user ' + context.currentKnownSubject.uniqueId + ' is : ' + dob);
            if (dob && validateDOB(dob)) {
                var birthDate = new Date(dob);
                if (getAge(birthDate) >= ageLimit) {
                    underAge = false;
                }
            }
            if (underAge === true) {
                Log.debug('User ' + context.currentKnownSubject.uniqueId + ' is under aged. Hence denied to login.');
                sendError(errorPage, errorPageParameters);
            }
        }
    });
};

var getAge = function(birthDate) {
    var today = new Date();
    var age = today.getFullYear() - birthDate.getFullYear();
    var m = today.getMonth() - birthDate.getMonth();
    if (m < 0 || (m === 0 && today.getDate() < birthDate.getDate())) {
        age--;
    }
    return age;
};
```