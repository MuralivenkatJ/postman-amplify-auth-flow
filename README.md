# AWS Cognito SRP Authentication in Postman

A complete, working Postman collection that fully replicates the `aws-amplify` (Cognito SRP) authentication flow. 

If you are developing backend APIs secured by AWS Cognito, fetching fresh access tokens usually means spinning up a frontend app or using the Cognito Hosted UI just to copy-paste tokens. This collection handles the complex Secure Remote Password (SRP) cryptography entirely within Postman Pre-request scripts, allowing you to fetch and automatically set your tokens in seconds.

## 🚀 Features
* **Zero Dependencies:** No external scripts or node server required. All math is handled using Postman's built-in `crypto-js` library and native `BigInt`.
* **Token Automation:** Automatically captures the `IdToken`, `AccessToken`, and `RefreshToken` upon successful login and saves them to your Postman Environment variables.
* **100% Amplify Parity:** Perfectly mirrors the `amazon-cognito-identity-js` cryptographic flow, including AWS's undocumented backend quirks.

## ⚙️ Prerequisites: AWS Cognito Setup

For this script to work, your AWS Cognito **App Client** must be configured exactly like a frontend web application:

1. **Enable SRP Auth:** In your User Pool App Client settings, ensure `ALLOW_USER_SRP_AUTH` is enabled in the Authentication Flows.
2. **NO Client Secret:** The App Client must **NOT** have a client secret generated. (Frontend apps cannot securely store secrets, and this script does not generate the `SECRET_HASH` required for secret-enabled clients).

## 🛠️ Quick Start Guide

1. **Import the Files:** Import the `AWS Amplify Cognito Signin.postman_collection.json` and the `AWS Amplify Cognito Signin ENV.postman_environment.json` files into Postman.
2. **Select the Environment:** Make sure the imported environment is set as your active environment in the top right corner of Postman.
3. **Fill in your Variables:** Update the environment with your specific AWS credentials:
   * `region`: Your AWS region (e.g., `us-east-1`, `ap-south-1`).
   * `userpoolId`: Your full User Pool ID (e.g., `us-east-1_Adfg12345`).
   * `clientId`: Your App Client ID (e.g., `38fjsnc484p94kpqsnet7mpld0`).
   * `username`: The user's email, phone number, or username.
   * `password`: The user's password.

4. **Run the Requests:**
   * Run **API Call 1 (Initiate Auth)** to begin the SRP handshake and fetch the challenge parameters.
   * Run **API Call 2 (Complete Sign In)** to calculate the final signature, respond to the challenge, and fetch your tokens.

Your `{{accessToken}}` and `{{idToken}}` are now saved in your environment variables and ready to be used as Bearer tokens in your other API requests!

## 🧠 How It Works (The "Magic" Explained)

Replicating AWS Cognito's SRP flow is notoriously difficult because it deviates from standard SRP implementations in a few undocumented ways. If you are looking through the Pre-request scripts and wondering why certain things are hardcoded, here is why:

* **The Massive `N_HEX` Variable:** This is not a secret key. It is the **3072-bit SRP Safe Prime** defined by the IETF in RFC 5054 (Group 15). It is a public mathematical constant required for the modular exponentiation that generates your public keys. It must remain exactly as it is.
* **The Java `BigInteger` Padding Quirk:** Cognito's backend is written in Java. Java's `BigInteger` uses Two's Complement, meaning any hex string starting with 8-F is interpreted as a negative number. The `padHex()` function in this script intercepts these values and prepends a `00` byte to force the number to remain positive, mirroring Java's behavior.
* **`Caldera Derived Key`:** When AWS passes the shared secret through the HKDF algorithm to generate the final key, it uses the hardcoded info string `"Caldera Derived Key"`. This was likely an internal Amazon project codename for this authentication flow that got permanently baked into the production architecture. 

## ⚠️ Security Warning
This collection requires storing plaintext passwords in Postman variables. **Never commit your active Postman Environment file to version control.** Always use the dummy template provided when sharing your setup.
