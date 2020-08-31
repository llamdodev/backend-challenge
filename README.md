
# backend-challenge

This repository is a first touch at a backend service to serve the data for Waldo.

We describe the needs of a client of this API. Your goal is to write the service, by implementing all the endpoints described below.

Questions are always welcome at developers@waldo.io

If you're wondering what this API is actually used for, you can have a look at https://github.com/waldoapp/frontend-challenge
for our end goal.

## Submit

Please add [@laurentsigal](https://github.com/laurentsigal) as a contributor to your github repository.

Otherwise you can simply host the code anywhere and give us a link to the zip file.

Do not make your repository public.

# Guidelines

The goal of this challenge is to see how you would structure a simple API service.

We require it to be written in Node (12+ is better). This is what we use thoroughly at Waldo, and even though it might not be your favorite language at the time, this is what you would end up using at Waldo.

Please do not use a complex framework in order to solve for this. It tends to provide too much functionality for free, which defies the purpose of the challenge. It also makes the code very hard to read in the absence of knowledge of said framework.

Packages are always encouraged of course (http frameworks like koa or express ; db connectors ; etc..).

Checklist:
- you should write the code in node
- favor a relational database
- We should be able to run the code on our end in a few steps. Please provide a comprehensive README
- We tend to pay attention to the organization of code
- (bonus) Provide testing
- (bonus) Provide a section of how you would deploy this code
- (bonus) How would you handle the failure scenario for comparisons? (e.g. no PATCH request is ever received)

# Flow of actions

The client needs to:
- create an application object, corresponding to an entity in the App Store.
- upload new versions of this application
- compare existing versions of the application together

### Create application
```
POST /applications
{
  identifier: { some identifier }
}
```
should return
```
{
  id: { unique id for the app in our system }
  identifier: { same as passed in }
}
```
The only requirement here is that the identifier should be defined and unique among all applications in our
system.

### Add a new version

Once an application has been created, the client should be able to upload new versions for it.
Versions have a status attached, which can be:
- **new** the version was just uploaded
- **analyzed** the version was compared

```
POST /applications/aId/versions
{
  checksum: { check sum of the binary file uploaded }
}
```
should return
```
{
  id: { unique id for the version in our system }
  applicationId: { foreign key to an application }
  versionNumber: { count of the version in this application to be able to replay the history }
  checksum: { same as passed in }
  status: "new"
}
```

Requirements:
- we want to number the versions from 1 to N as they are uploaded for the application
- this number must be unique per application, and it is likely that 2 uploads happen at the same time for the same application
- a version always has the status `new` when uploaded

### Create a comparison

A comparison takes 2 versions of the same app, and structures the differences between them.
Such structure for the differences is irrelevant to the challenge here (but you can get an idea of how that
looks by checking [the frontend-challenge](https://github.com/waldoapp/frontend-challenge)).

Since it takes a long time, a comparison is first initialized (with status = `pending`), and then
it is updated with the proper status from [`success`, `error`].

```
POST /applications/aId/comparisons
{
  headVersionId: { id of the "recent" version }
  baseVersionId: { id of the "base" version, i.e. the one taken as a reference }
}
```
should return
```
{
  id: { unique id for the version in our system }
  applicationId: { foreign key to an application }
  headVersionId: { id of the "recent" version }
  baseVersionId: { id of the "base" version, i.e. the one taken as a reference }
  status: "pending"
}
```

Requirements:
- a comparison should always compare a recent version (`head`) with an earlier one (`base`)
- a comparison should always compare with `base` being already analyzed (with one obvious exception).
- we do not allow several running comparisons for the same application at the same time
- we prevent the same 2 versions to be compared several times together

### Finalize a comparison

At the end of the actual comparison, the client can finalize a comparison with a status
```
PATCH /comparisons/cId
{
  status: { either "success" or "error" }
}
```
should return
```
{
  id: { unique id for the version in our system }
  applicationId: { foreign key to an application }
  headVersionId: { id of the "recent" version }
  baseVersionId: { id of the "base" version, i.e. the one taken as a reference }
  status: { same as passed in }
}
```

Rules:
- only the status can be edited for a build
- once the status of a build is "success" or "error", it becomes immutable
- this should update the underlying versions to the proper status
