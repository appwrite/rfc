# Response Model in SDKs for improved DX <!-- What do you want to call your `awesome_feature`? -->

- Implementation Owner: @lohanidamodar
- Start Date: 02-25-2021
- Target Date: N/A
- Appwrite Issue: N/A

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
Having proper response model while making an API request would improve developer experience a lot. This will make working with Appwrite SDKs a lot easier.

## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

<!-- Write your answer below. -->
At the moment, all of the SDKs return JSON response which then have to be parsed by the developers to make proper use. Instead, having a proper response model and returning response object instead of JSON would make developers tasks a lot easier. They will not have to make their own response model for each and every endpoints, they will get proper response objects that they can use natively.

**What is the context or background in which this problem exists?**

<!-- Write your answer below. -->
All of Appwrite's endpoints return JSON objects, which can be parsed into native models and accessed. However

**Once the proposal is implemented, how will the system change?**

<!-- Write your answer below. -->

The Appwrite itself already provides the Response object in the Swagger specification. So it will not change. However, All the SDKs will change, each functions will return proper response object.

<!-- Please avoid discussing your proposed solution. -->

## Design proposal (Step 2)

[design-proposal]: #design-proposal

<!--
This is the technical portion of the RFC. Explain the design in sufficient detail keeping in mind the following:

- Its interaction with other parts of the system is clear
- It is reasonably clear how the contribution would be implemented
- Dependencies on libraries, tools, projects or work that isn't yet complete
- New API routes that need to be created or modifications to the existing routes (if needed)
- Any breaking changes and ways in which we can ensure backward compatibility.
- Use Cases
- Goals
- Deliverables
- Changes to documentation
- Ways to scale the solution

Ensure that you include examples, code-snippets etc. to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**

Write your answer below.

-->

SDK templates should now should also consider the response object and response schema defined in the Swagger specification. We need to get those schemas and response definitions from Swagger to SDK templates and perpare a proper response model.

## 1. Define models
Define model of all available response objects. From Swagger spec, we can get the list of all the available response objects in `definitions` object where each response object is `key=>value` paired where key is the name of the response object and value the definition of all the fields. For each of these response object, in each SDK we should create models.

## 2. Return object instead of JSON
Once we have all the models, for each API end points based on the response definitions (available in `responses` object under `schema`) in the Swagger, we should convert the JSON to corresponding object and return.

For each SDK the process of converting JSON to respective model might be different, so we should work accordingly.

Below we will see an example from Dart/Flutter SDK. Let's look at create user endpoint. According to Swagger definitions, it returns User on successful request, so first we create user model as follows.

```dart
class User {
    User({
        this.id,
        this.name,
        this.registration,
        this.status,
        this.email,
        this.emailVerification,
        this.prefs,
    });

    String id;
    String name;
    int registration;
    int status;
    String email;
    bool emailVerification;
    String prefs;

    factory User.fromJson(Map<String, dynamic> json) => User(
        id: json["\$id"],
        name: json["name"],
        registration: json["registration"],
        status: json["status"],
        email: json["email"],
        emailVerification: json["emailVerification"],
        prefs: json["prefs"],
    );

    Map<String, dynamic> toJson() => {
        "\$id": id,
        "name": name,
        "registration": registration,
        "status": status,
        "email": email,
        "emailVerification": emailVerification,
        "prefs": prefs,
    };
}
```

So now the `create` user function,

```dart
Future<User> create({@required String email, @required String password, String name = ''}) async {
    final String path = '/users';

    final Map<String, dynamic> params = {
        'email': email,
        'password': password,
        'name': name,
    };

    final Map<String, String> headers = {
        'content-type': 'application/json',
    };

    final res = await client.call(HttpMethod.post, path: path, params: params, headers: headers);
    return User.fromJson(res.data);
}
```

### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal. A
few examples of what this can include are:

- Does this functionality exist in other software and what experience has their
  community had?
- For other teams: What lessons can we learn from what other communities have
  done here?
- Papers: Are there any published papers or great posts that discuss this? If
  you have some relevant papers to refer to, this can serve as a more detailed
  theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us
whether they are brand new or if it is an adaptation from other software.

Write your answer below.
-->
May popular SDKs for popular softwares, always return proper response objects instead of plain String or JSON. This improves developer experience a lot. A developer can easily understand, what object the API is returning, what methods and properties are available.

- Owlbot dart SDK - (https://pub.dev/packages/owlbot_dart) is one example where the SDK returns properly formatted response objects instead of plain JSON

- GitHub unofficial dart SDK - (https://pub.dev/packages/github) is another example where SDK returns properly formatted response objects.

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->
1. What to do with response objects that contain dynamic fields like documents (defined as any in the swagger spec), user prefs etc
2. Handling serialization and dserialization in each SDKs by keeping things similar between SDKs as well as keeping the native feel of the SDKs as it is.

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->

1. how to handle object properties
2. Display response objects on the appwrite docs
3. Fromjson/tojson methods (serialization/dserialization)