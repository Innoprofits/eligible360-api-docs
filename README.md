# Eligible 360 - API V1 Documentation

# Introduction

Eligible360.ca offers a REST API that gives developers the possibility to integrate its recommendation engine with the clients custom platforms.

The API is private and only accessible by customers of Eligible360.ca.

# Summary
1. Getting started
2. Managing accesses
    1. List all available accesses
    2. Show single access
    3. Create an access
    4. Update an access
    5. Delete an access
3. Managing entries
    1. List all entries
    2. Show an access' latest entry
    3. Download PDF report of an entry
    4. Create a new entry

4. Miscellaneous
    1. Terms and conditions
    2. Privacy policy

# Getting started

To get started, developers must obtain an access token from the Eligible360.ca customer dashboard.

An access token is related to a single survey, and can be generated from the Survey details page.

All API calls are scoped to a survey, using the survey's unique key. Example:

```javascript
const survey_key = "ABCdef";
const bearer_token = "...";
fetch(`https://eligible360.ca/api/v1/${survey_key}/...`, {
    method: '...',
    headers: {
        'Accept': 'application/json',
        'Authentication': `Bearer ${bearer_token}`
    },
    body: {
        //...
    }
})
```

# Managing accesses

An "access" is an account of a single person or entity that will answer the questions in order to get recommendations about programs.

An "access" has an "entry" that contains the answers to the questions, along with the eligible programs depending on them.

For a new user to start answering questions, an "access" has to be created for him.

## List all available accesses

URL: `/accesses`

Method: `GET`

### Success response

Code: `200 OK`

Content example:

```json
{
   "data": [
      {
         "id": 1,
         "username": "johndoe@example.org",
         "name": "John Doe",
         "organization": "ACME inc",
         "phone": null,
         "created_at": "2021-10-10 10:10:10"
      },
      //...
   ]
}
```

## Show single access

URL: `/accesses/{id}`

Method: `GET`

### Success response

Code: `200 OK`

Content example:

```json
{
   "data": {
      "id": 1,
      "username": "johndoe@example.org",
      "name": "John Doe",
      "organization": "ACME inc",
      "phone": null,
      "created_at": "2021-10-10 10:10:10"
   }
}
```

## Create an access

URL: `/accesses`

Method: `POST`

Payload:

```json
{
   "username": "johndoe@example.org", // Required, and unique per survey
   "name": "John Doe", // Optional
   "organization": "ACME inc", // Optional
   "phone": null // Optional
}
```

### Success response

Code: `201 Created`

Content example:

```json
{
   "data": {
      "id": 1,
      "username": "johndoe@example.org",
      "name": "John Doe",
      "organization": "ACME inc",
      "phone": null,
      "created_at": "2021-10-10 10:10:10"
   }
}
```

### Error response

Code: `422 Unprocessable Entity`

Reason: Validation error

## Update an access

URL: `/accesses/{id}`

Method: `PUT`

Payload:

```json
{
   "username": "johndoe@example.org", // Required, and unique per survey
   "name": "John Doe", // Optional
   "organization": "ACME inc", // Optional
   "phone": null // Optional
}
```

### Success response

Code: `200 OK`

Content example:

```json
{
   "data": {
      "id": 1,
      "username": "johndoe@example.org",
      "name": "John Doe",
      "organization": "ACME inc",
      "phone": null,
      "created_at": "2021-10-10 10:10:10"
   }
}
```

### Error response

Code: `422 Unprocessable Entity`

Reason: Validation error

## Delete an access

URL: `/accesses/{id}`

Method: `DELETE`

### Success response

Code: `200 OK`

# Managing entries

An "entry" is created when an "access" answers all questions, and the recommendation engine returns a list of programs.

## List all entries

URL: `/entries`

Method: `GET`

### Success response

Code: `200 OK`

Content example:

```json
{
   "data": [
      {
         "access": {
            "id": 1,
            "username": "johndoe@example.org",
            "name": "John Doe",
            "organization": "ACME inc",
            "phone": null,
            "created_at": "2021-10-10 10:10:10"
         },
         "created_at": "2021-10-10 11:11:11"
      },
      //...
   ]
}
```

## Create a new entry

URL: `/accesses/{id}/entry`

Method: `POST`

Payload:

Depending on the payload, the endpoint can execute multiple operations:

### Create a new entry from zero

```json
{
   "answers": [] // Empty array
}
```

### Success response

Code: `200 OK`

Content example:

```json
{
  "status": "form",
  "data": {
    "condition": {
      "id": "G5",
      //...
    },
    "progress": 0
  }
}
```

The returned payload contains the following:
* `status`: If present, and its value is `form` means there are still questions to be answered.
* `data` object contains the condition (or the question) to be answered with its key (or id)
* `data` object also contains the progress of the survey

### Answering questions

The answer is an object containing two items:
* The key (id) of the question returned by the endpoint
* The value of the answer for the question, can be 0, 1, a string, or an array of strings, depending on the type of the question

The new answer should be pushed to the answers array, and sent back to the endpoint

```json
{
   "answers": [
      // Previous answers objects
      {
         "key": "G5",
         "value": "1632320776815"
      }
   ]
}
```

The payload sent to the endpoint can contain a `program` parameter, this is used to confirm the eligibility of a user in a possible program (see below) by passing its id in the parameter

```json
{
   "programs": 1,
   "answers": [
      // All previous answers
   ]
}
```

If all questions are answered, an entry is returned with the answers along with the programs.

If the user is not eligible to any program, a payload with `status: no-programs` is returned
```json
{
   "status": "no-programs"
}
```

## Show an access' latest entry

URL: `/accesses/{id}/entry`

Method: `GET`

### Success response

Code: `200 OK`

Content example:

```json
{
   "data": {
      "access": {
         "id": 1,
         "username": "johndoe@example.org",
         "name": "John Doe",
         "organization": "ACME inc",
         "phone": null,
         "created_at": "2021-10-10 10:10:10"
      },
      "created_at": "2021-10-10 11:11:11",
      "answers": [
         {
            "key": 23,
            "value": 1
         },
         {
            "key": "G1",
            "value": "2010"
         },
         {
            "key": "G2",
            "value": [
               "A",
               "B",
               "C"
            ]
         }
         //...
      ],
      "programs": [
         {
            "id": 1,
            "title": "Aménagement et réduction du temps de travail",
            "details": "...",
            "link": "https://...",
            "deadline": "...",
            "date": "...",
            "type": "Subvention",
            "application_process": "...",
            "expert_advice": "..."
         },
         //...
      ],
      "possible_programs": [
         {
            "id": 1,
            "title": "Aménagement et réduction du temps de travail",
            "details": "...",
            "link": "https://...",
            "deadline": "...",
            "date": "...",
            "type": "Subvention",
            "application_process": "...",
            "expert_advice": "..."
         },
         //...
      ]
   }
}
```

The returned payload contains the following:

* The access associated with the entry
* Array of answers provided for said entry
* Array of eligible programs, each program is an object containing details
* Array of programs that have a possibility of eligibility if the user answers few more questions specific to the program in question

## Download PDF report of an entry

URL: `/accesses/{id}/entry/pdf`

Method: `GET`

### Success response

Code: `200 OK`

Content: `application/pdf`
