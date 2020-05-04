.. _api:

REST API
========

--------------
Authentication
--------------

TPA Stream's REST API supports Basic Authentication over HTTPS using your API Key.

To implement, simply put your user name (email address) and API key in the Authorization header on each API request.

**Your API key is a Secret Password! Do not share this key with anyone.**

A simple example using curl:

.. code-block:: bash

    curl -L --user me@example.com:MY_API_KEY https://app.tpastream.com/api/claims

-------------------
API Response Format
-------------------

Each API endpoint, upon success, will return an object with the entity (or entities) in the data key.
If the endpoint is paginated, it will also contain a number of other pagination related fields:

.. code-block:: javascript

    {
      "data": [],  // or {} for a single entity
      "has_next": true,
      "has_prev": false,
      "next_num": 2,
      "page": 1,
      "pages": 123,
      "per_page": 1000,
      "prev_num": 0,
      "total": 122901
    }

----------
API Errors
----------


In the event that there is an error, our response will have a key of message.
It may contain some other additional information depending on the nature of the error.
If the error is an unexpected error (and raises an exception in our system),
it will also contain an event_id. If you see an event_id, then our engineers
are notified that there is an issue.

We make every effort to provide meaningful HTTP status codes and messages when
there is an error. Please do not hesitate to contact us regarding your usage of
our API by emailing support@tpastream.com.

Error responses will look something like this:

.. code-block:: json

    {
      "message": "There was an error and hopefully this is useful info.",
      "event_id": "xxxxxxxxxx"
    }


------------------------
A note about Date Ranges
------------------------

Types of data that are date ranges are serialized to JSON as an object with bounds:

.. code-block:: json

    {
      "bounds": "[)",
      "end": "2016-09-16",
      "start": "2016-09-15"
    }

Note that since date of service can span over multiple days
(for example, a hospital stay), it is stored as a range.

    *In the text form of a range, an inclusive lower bound is represented by "[" while an exclusive lower bound is represented by "(". Likewise, an inclusive upper bound is represented by "]", while an exclusive upper bound is represented by ")"*

    -- For more details and examples, please consult the PostgreSQL documentation https://www.postgresql.org/docs/9.6/static/rangetypes.html#RANGETYPES-INCLUSIVITY

------------------------------------
Filtering and Paging Through Results
------------------------------------

Most API endpoints have enforced pagination. There are currently no globally
defined upper limits to how much can be requested per_page, but please try to
maintain a healthy balance between number of requests and amount of data requested
via per_page with each request. When pulling large amounts of data, we recommend
starting with per_page=1000 and optimizing as necessary. Please contact
support@tpastream.com if you intend to pull large amounts of data, as we can
help define a strategy or custom endpoint to better serve both your needs & ours.

Here are a list of endpoints. GET, PUT, POST, and DELETE are supported for most
endpoints, however may or may not be enabled for each type of user.

**GET All Claims (starts with page 1. The response will tell you if there are more pages available)**

::

    /api/claims

**GET All Employers**

::

    /api/employer

**GET All Members**

::

    /api/member

**GET All Policy Holders**

::

    /api/policy_holder

**GET All Claims where Employer ID is 99999**

::

    /api/employer/99999/claims

**GET All Claims where Policy Holder ID is 99999**

::

    /api/policy_holder/99999/claims

**GET All Claims where Member ID is 99999**

::

    /api/member/99999/claims

**GET Page 3 of Claims with 1000 Claims per page for Employer 999**

::

    /api/employer/999/claims?per_page=1000&page=3

**GET Page 3 of Claims with 10 Claims per page for Employer 99. Do not include claims that are marked as “Read”, and do not include claims before the Employer’s “Effective Date”**

::

    /api/employer/99/claims?per_page=10&page=3&hide_read=on&hide_before_effective_date=on

**Claims Response**

.. code-block:: javascript

    {
      "data": [
         {
            "amount_allowed": null, 
            "amount_billed": 4509.00, 
            "amount_not_covered": null, 
            "amount_paid": 487.90, 
            "amount_paid_other": null, 
            "check_date": null, 
            "check_number": null, 
            "claim_medical_id": 9999999, 
            "claim_medical_lines": [
            {
               "amount_allowed": 69.11, 
               "amount_billed": 85.00, 
               "amount_not_covered": null, 
               "amount_paid": 0.00, 
               "amount_paid_other": null, 
               "claim_medical_line_id": 999999, 
               "coinsurance_patient": 0.00, 
               "copayment": 0.00, 
               "date_of_service": {
                  "bounds": "[)", 
                  "end": "2016-10-26", 
                  "start": "2016-10-25"
               }, 
               "diagnosis_code": null, 
               "discount": null, 
               "patient_responsibility": null, 
               "polymorphic__amount_allowed": null, 
               "polymorphic__amount_billed": null, 
               "polymorphic__amount_paid": null, 
               "polymorphic__coinsurance_patient": null, 
               "polymorphic__copayment": null, 
               "polymorphic__patient_responsibility": null, 
               "polymorphic__reduction": null, 
               "procedure_code": null, 
               "procedure_name": "Office/outpatient Visit, Est", 
               "reduction": 69.11, 
               "total_patient_responsibility": 69.11, 
               "vendor_system_id": "0"
            }
            ],
            "claim_requests": [], 
            "coinsurance_patient": 209.10, 
            "copayment": 0.00, 
            "createddate": "2017-05-28T06:47:16.361817-04:00",
            "dataobject_id": 9999,
            "date_of_service": null,
            "dependents": [
            {
               "alegeus_key": null, 
               "createddate": "2018-03-29T08:47:12.044480-04:00", 
               "datapath_key": null, 
               "email": null, 
               "first_name": "Johnny", 
               "generic_key": null, 
               "id": 99999, 
               "last_name": "Appleseed", 
               "modifieddate": "2018-03-29T08:47:12.044480-04:00", 
               "ssn": null, 
               "wex_key": null
            }
            ],
            "discount": null,
            "eob_date": null, 
            "group_name": null, 
            "group_number": null,
            "id": 476877,
            "last_updated_status": "2017-05-28T06:47:16.361817-04:00", 
            "members": [
            {
               "email": "johnny@appleseed.com", 
               "employer_id": 99999, 
               "employer": {
                  "id": 99999, 
                  "name": "Fruit Tree Planting Services, LLC", 
                  "reimbursement_policy": "off"
               },

               "full_name": "Johnny Appleseed", 
               "id": 888888
            }
            ], 
            "modifieddate": "2017-05-28T06:47:16.361817-04:00", 
            "network": null, 
            "patient_account_number": null, 
            "patient_name": "Jimmy Appleseed", 
            "patient_responsibility": 3959.10, 
            "policy_holder": {
            "fullname": "Johnny Appleseed", 
            "policy_holder_id": 888888
            }, 
            "policy_holder_fullname": "Johnny Appleseed", 
            "policy_holder_id": 888888, 
            "polymorphic__amount_allowed": null, 
            "polymorphic__amount_billed": null, 
            "polymorphic__amount_paid": null, 
            "polymorphic__coinsurance_patient": null, 
            "polymorphic__copayment": null, 
            "polymorphic__patient_responsibility": null, 
            "polymorphic__reduction": null,
            "processed_on": "2016-10-15", 
            "read": [], 
            "read_all": [], 
            "reduction": 0.00, 
            "remarks": null, 
            "service_provider": "Dr. Suess", 
            "service_provider_address": null, 
            "service_provider_billing_address": null, 
            "service_provider_billing_name": null, 
            "service_provider_billing_npi_number": null, 
            "service_provider_billing_number": null, 
            "service_provider_billing_phone": null, 
            "service_provider_npi_number": null, 
            "service_provider_number": null, 
            "status": "Processed", 
            "total_patient_responsibility": 69.11, 
            "status": "Processed", 
            "tpafiles": [
            {
               "extension": ".png", 
               "tpafile_id": 99999, 
               "url": "/claim_medical/99999/tpafile/88888"
            }, 
            {
               "extension": ".pdf", 
               "tpafile_id": 44444, 
               "url": "/claim_medical/99999/tpafile/88888"
            }
            ], 
            "type": {
            "name": "dental", 
            "type_id": 2
            }, 
            "vendor_system_id": "xxxxx122344"
         }
      ], 
      "has_next": true, 
      "has_prev": false,
      "next_num": 2, 
      "page": 1, 
      "pages": 9999, 
      "per_page": 1, 
      "prev_num": 0, 
      "total": 9999
      }


**Employer Response**

.. code-block:: javascript

    {
      "data": [
         {
            "accounts": [], 
            "alegeus_key": null, 
            "can_request_reimbursements": false, 
            "can_use_portal": false, 
            "createddate": "2016-12-10T12:17:09.497104-05:00", 
            "datapath_key": "99999", 
            "easy_enroll_ssn_required": true, 
            "effective_date": "2016-05-01", 
            "email_automation": true, 
            "employer_id": 5555555, 
            "generic_key": null, 
            "is_demo": false, 
            "modifieddate": "2017-02-28T18:07:03.799519-05:00", 
            "name": "Dunder Mifflin Paper Company", 
            "onboard_field_send_reimbursement": "all", 
            "onboard_url": "https://www.easyenrollment.net/enroll/ddddd", 
            "payers": [
            {
               "logo_url": "https://s3.amazonaws.com/tpastream-public/HorizonBlue-Logo-Updated-Jan15.jpg", 
               "name": "Horizon Blue Cross Blue Shield of New Jersey", 
               "payer_id": 33, 
               "retriever": "horizon_bluecross.HorizonBlue", 
               "short_name": "Horizon BCBS NJ"
            }
            ], 
            "send_new_claim_emails": false, 
            "slug": "dddd", 
            "support_email": null, 
            "support_email_derived": "support@my-tpa.com", 
            "support_phone": null, 
            "support_phone_derived": "(800) 999-9999", 
            "team_primary": null, 
            "team_primary_id": null, 
            "teams": [], 
            "tenant": {
            "logo_url": "https://s3.amazonaws.com/tpastream-public/xxxxxxx.png", 
            "name": "My TPA", 
            "tenant_id": 99999
            }, 
            "unread_count": 333, 
            "wex_key": null
         }

      ], 
      "has_next": true, 
      "has_prev": false, 
      "next_num": 2, 
      "page": 1, 
      "pages": 66, 
      "per_page": 1, 
      "prev_num": 0, 
      "total": 66
      }
