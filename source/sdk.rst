.. _sdk:

JavaScript SDK
==============

--------------------
Why a JavaScript SDK
--------------------

The purpose for using the JavaScript SDK provides a seamless enrollment
experience for members. While other methods such as SAML2 SSO to simplify
the enrollment experience work well,  as opposed to other methods of
enrollment (such as SAML2 SSO), is to provide a more seamless enrollment
experience for Members.


----------
Philosophy
----------

This SDK is designed to implement the EasyEnrollment platform into our
clients own hosted web-portals. We want to make it fit as seamlessly as
possible with the current experience of their sites. As such, we have
provided functionality to add callbacks to the end of each of the necessary
flows and we are as un-opinionated as possible about the styling of the SDK's
flow.


------------
Client Usage
------------


***********
NPM Example
***********

.. code-block:: javascript

    npm i easyenrollsdk
 
    import TPAStream_EasyEnroll from 'easyenrollsdk';
   
    const sdk = TPAStream_EasyEnroll({
        el: '#react-hook',
        isDemo: true
    });


***********
CDN Example
***********
.. code-block:: html

    <script src="https://app.tpastream.com/static/js/sdk.js"></script>
    <script>
        window.TPAStream_EasyEnroll({
            el: '#react-hook',
            employer: {
                systemKey: 'some-system-key',
                vendor: 'internal',
                name: 'some-name'
            },
            user: {
                firstName: 'Joe', 
                lastName: 'Sajor', 
                email: 'some-email@place.com',
                memberSystemKey: 'some-system-key',
                phoneNumber: '333333333',
                dateOfBirth: '11-11-1121' 
            },
            apiToken: 'Some Provided Key → 21poi34kjqf21j1poi1d2po', // We'll provide this.
            realTimeVerification: true,
            doneChoosePayer: () => {},
            doneTermsOfService: () => {},
            doneCreatedForm: () => {},
            doneRealTime: () => {},
            donePopUp: () => {},
            doneEasyEnroll: ({ employer, payer, tenant, policyHolder, user }) => {},
            handleFormErrors: (error, {response, request, config}) => {},
            userSchema: {}
        })
    </script>


--------------------
Supported Parameters
--------------------

The SDK currently supports the following parameters:

* :code:`el` (This is where the SDK will render: Note -> This is a ‘css selector’)
* :code:`employer`

    * :code:`systemKey`
    * :code:`vendor` (This will usually be 'internal')
    * :code:`name`
* :code:`user`

    * :code:`firstName`
    * :code:`lastName`
    * :code:`email`
    * :code:`memberSystemKey`
    * :code:`phoneNumber`
* :code:`apiToken`
* :code:`realTimeVerification` -> Bool
* :code:`isDemo` -> Bool (This is recommended for sandboxing before you hook the SDK up for real)
* :code:`userSchema` (This is an object {} following react-jsonschema-form for making ui:schema)
* :code:`doneChoosePayer` *
* :code:`doneTermsOfService` *
* :code:`doneCreatedForm` *
* :code:`donePopUp` *
* :code:`doneRealTime` *
* :code:`doneEasyEnroll` * (Below are args passed into the func)

  * :code:`employer`
  * :code:`payer`
  * :code:`policyHolder`
  * :code:`user`
  * :code:`tenant`
* :code:`handleFormErrors` *

  * :code:`error`
  * :code:`error_parts`

    * :code:`response`
    * :code:`request`
    * :code:`config`

(Required parameters are Highlighted: Note only ‘el’ is required for demo mode)

Function (() => {}) parameters are Starred*
