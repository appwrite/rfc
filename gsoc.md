# Appwrite GSoC 2023 Project Ideas
Below are project ideas for GSoC 2023. Each of these problem statements will go through a full 
[RFC process](https://github.com/appwrite/rfc) process like all other contributions to Appwrite.
If you have questions, ideas, or concerns, feel free to find us on [Discord](https://appwrite.io/discord)!

Mentors listed are subject to change based on timezone, but we'll keep you updated.

## Project Ideas
### Appwrite UI Packages
- Description: Create UI helper libraries to help developers implement authentication, realtime, and other Appwrite integrations with less code. The UI helper library can be for Web, Flutter, iOS, or Android platforms. Some inspiration can be found from Supabase’s auth-helper,  Firebase firebase_ui_auth and Firebase UI for Web. Appwrite’s Pink design system can be used as inspiration for UI.
- Expected Outcome: 
  - A UI helper library for one of the client platforms. 
  - The UI helper library should cover components for all authentication methods. 
  - (Optional) Additional components can be created for other Appwrite services, like Database, Storage, Functions, or Realtime, but is not strictly necessary.
- Recommended Skills: One of: Web, iOS, Android, or Flutter development.
- Mentor(s):
  - @abnegate
  - @lohanidamodar
- Expected project size: 175 Hours
- Difficulty: Easy
- Upstream Issue (URL): N/A, be creative!
### S3 Generic Integration
- Description: Create a generic S3 storage adaptor for Appwrite. This adaptor can be configured to create connection to any S3 compatible storage service, such as MinIO, AWS S3, DigitalOcean Spaces, Wasabi, etc.
- Expected Outcome: A generic S3 adaptor for Appwrite that can be connected to any S3 compatible storage service.
- Recommended Skills: Knowledge in one of the following languages ( JavaScript, Python, C++ or Java ), S3, Docker
- Mentor(s):
  - @lohanidamodar
- Expected project size: 350 Hours
- Difficulty: Hard
- Upstream Issue (URL): 
  - https://github.com/utopia-php/storage/issues/28#issuecomment-1373955806 
### Kubernetes Support
- Description: To date, Appwrite runs as a set of microservices using Docker. This allows Appwrite to be deployed easily without having to worry about dependencies on the operating system. Docker (via the docker socket) is also used to spin up Functions at runtime. In addition to running on Docker, it would be helpful if Appwrite can run on Kubernetes (K8s), a leading system for automating, deploying, scaling, and managing containerized applications.
- Expected Outcome: 
  - A Helm Chart that can be used to deploy Appwrite on Kubernetes. 
  - All functionality of Appwrite works including Storage and Functions.
- Recommended Skills: Knowledge in one of the following languages ( JavaScript, Python, C++ or Java ),  Docker, Kubernetes
- Mentor(s):
  - @christyjacob4
- Expected project size: 350 Hours
- Difficulty: Hard
- Upstream Issue (URL):
  - https://github.com/appwrite/appwrite/issues/24
  - https://github.com/appwrite/appwrite/issues/364

### New DB Adaptor
- Description: Create a new DB adaptor, similar to Appwrite’s PostgreSQL or MongoDB adaptors. Allow Appwrite to be backed by a new database. Some suggested options are: Oracle, Microsoft SQL Server, IBM DB2, Couch DB, Cassandra, or another popular database.
- Expected Outcome: 
  - A new database adaptor for Utopia PHP
  - Allow Appwrite to be configured to use this new DB adaptor
- Recommended Skills: SQL, Knowledge in one of the following languages ( JavaScript, Python, C++ or Java ) 
- Mentor(s):
  - @vermakhushboo
  - @wess
- Expected project size: 350 Hours
- Difficulty: Hard
- Upstream Issue (URL): N/A
  
### Helper Libraries for Open Runtimes
- Description: Appwrite has cloud function runtimes in over 8 different languages. Each of these runtimes use a Request, Response and more objects to facilitate a good developer experience. However, we are missing the implementation of the Request and Response classes as they are native to the runtime server. 
- Expected Outcome: Implement the Request and Response classes as standalone libraries that can be imported into the respective runtimes and offer an improved developer experience.
- Recommended Skills: Knowledge in one of the following languages ( JavaScript, Python, C++ or Java ) 
- Mentor(s):
  - @Meldiron
  - @christyjacob4
- Expected project size: 175 Hours
- Difficulty: Medium
- Upstream Issue (URL): N/A, get creative!

  
### Multiple Generic OAuth OIDC providers
- Description: Appwrite supports dozens of OAuth providers. However, at the moment, you can only have one of each type of OAuth provider. Some OAuth providers (e.g. Okta, Microsoft, Authentik, etc.) can have multiple tenants/configurations, but in Appwrite, you’d only be able to use one. Also, ideally, considering OIDC is pretty standardized, it should be possible to have a generic OIDC provider and use it with any Auth provider that supports OIDC.
- Expected Outcome: 
  - A generic OIDC provider which can be used with any auth provider that supports OIDC
  - Allow multiple of a single type of oauth provider
- Recommended Skills: Knowledge in one of the following languages ( JavaScript, Python, C++ or Java ), OAuth2, OIDC
- Mentor(s):
  - @TorstenDittmann
- Expected project size: 175 Hours
- Difficulty: Medium
- Upstream Issue (URL): 
    - https://github.com/appwrite/appwrite/issues/4299
  
### Authentication with WebAuthn
- Description: WebAuthn is a web-based authentication standard that enables secure authentication with public key cryptography. It provides a secure, passwordless login experience and enables users to authenticate to websites and applications using hardware-based security keys, biometric devices, or mobile devices. WebAuthn is designed to be interoperable, meaning that it works across different browsers, operating systems, and devices, providing a consistent and secure experience for users. In this project we’d like to add support for WebAuthn as an authentication method in Appwrite.
- Expected Outcome: 
  - Users are able to login to Appwrite using their hardware keys.
  - Create the necessary APIs in Appwrite to support WebAuthN
  - Create the necessary functionality in our Web SDKs to facilitate WebAuthn
  - Create a demo application to demonstrate the usage of those APIs
- Recommended Skills: Knowledge in one of the following languages ( JavaScript, Python, C++ or Java ) 
- Mentor(s):
  - @TorstenDittmann
  - @christyjacob4
- Expected project size: 350 Hours
- Difficulty: Medium
- Upstream Issue (URL): N/A
- Relevant Resources
  - https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API
  - https://webauthn.guide/
  - https://webauthn.io/
