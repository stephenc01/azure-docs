---
title: Set up sign-up and sign-in with a Twitter account
titleSuffix: Azure AD B2C
description: Provide sign-up and sign-in to customers with Twitter accounts in your applications using Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg

ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/07/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
---

# Set up sign-up and sign-in with a Twitter account using Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]
::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## Prerequisites

::: zone pivot="b2c-user-flow"

* [Create a user flow](tutorial-create-user-flows.md) to enable users to sign up and sign in to your application.
* If you haven't already done so, [add a web API application to your Azure Active Directory B2C tenant](add-web-api-application.md).

::: zone-end

::: zone pivot="b2c-custom-policy"

* Complete the steps in the [Get started with custom policies in Active Directory B2C](custom-policy-get-started.md).
* If you haven't already done so, [add a web API application to your Azure Active Directory B2C tenant](add-web-api-application.md).

::: zone-end

## Create an application

To use Twitter as an identity provider in Azure AD B2C, you need to create a Twitter application. If you don't already have a Twitter account, you can sign up at [https://twitter.com/signup](https://twitter.com/signup).

1. Sign in to the [Twitter Developers](https://developer.twitter.com/en/apps) website with your Twitter account credentials.
1. Select  **Create an app**.
1. Enter an **App name** and an **Application description**.
1. In **Website URL**, enter `https://your-tenant.b2clogin.com`. Replace `your-tenant` with the name of your tenant. For example, `https://contosob2c.b2clogin.com`.
1. For the **Callback URL**, enter `https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/your-user-flow-Id/oauth1/authresp`. Replace `your-tenant` with the name of your tenant name and `your-user-flow-Id` with the identifier of your user flow. For example, `b2c_1A_signup_signin_twitter`. You need to use all lowercase letters when entering your tenant name and user flow id even if they are defined with uppercase letters in Azure AD B2C.
1. At the bottom of the page, read and accept the terms, and then select **Create**.
1. On the **App details** page, select **Edit > Edit details**, check the box for **Enable Sign in with Twitter**, and then select **Save**.
1. Select **Keys and tokens** and record the **Consumer API Key** and the **Consumer API secret key** values to be used later.

::: zone pivot="b2c-user-flow"

## Configure Twitter as an identity provider in your tenant

1. Sign in to the [Azure portal](https://portal.azure.com/) as the global administrator of your Azure AD B2C tenant.
1. Make sure you're using the directory that contains your Azure AD B2C tenant by selecting the **Directory + subscription** filter in the top menu and choosing the directory that contains your tenant.
1. Choose **All services** in the top-left corner of the Azure portal, search for and select **Azure AD B2C**.
1. Select **Identity providers**, then select **Twitter**.
1. Enter a **Name**. For example, *Twitter*.
1. For the **Client ID**, enter the Consumer API Key of the Twitter application that you created earlier.
1. For the **Client secret**, enter the Consumer API secret key that you recorded.
1. Select **Save**.

::: zone-end

::: zone pivot="b2c-custom-policy"

## Create a policy key

You need to store the secret key that you previously recorded in your Azure AD B2C tenant.

1. Sign in to the [Azure portal](https://portal.azure.com/).
2. Make sure you're using the directory that contains your Azure AD B2C tenant. Select the **Directory + subscription** filter in the top menu and choose the directory that contains your tenant.
3. Choose **All services** in the top-left corner of the Azure portal, and then search for and select **Azure AD B2C**.
4. On the Overview page, select **Identity Experience Framework**.
5. Select **Policy Keys** and then select **Add**.
6. For **Options**, choose `Manual`.
7. Enter a **Name** for the policy key. For example, `TwitterSecret`. The prefix `B2C_1A_` is added automatically to the name of your key.
8. In **Secret**, enter your client secret that you previously recorded.
9. For **Key usage**, select `Encryption`.
10. Click **Create**.

## Add a claims provider

If you want users to sign in using a Twitter account, you need to define the account as a claims provider that Azure AD B2C can communicate with through an endpoint. The endpoint provides a set of claims that are used by Azure AD B2C to verify that a specific user has authenticated.

You can define a Twitter account as a claims provider by adding it to the **ClaimsProviders** element in the extension file of your policy.

1. Open the *TrustFrameworkExtensions.xml*.
2. Find the **ClaimsProviders** element. If it does not exist, add it under the root element.
3. Add a new **ClaimsProvider** as follows:

    ```xml
    <ClaimsProvider>
      <Domain>twitter.com</Domain>
      <DisplayName>Twitter</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Twitter-OAUTH1">
          <DisplayName>Twitter</DisplayName>
          <Protocol Name="OAuth1" />
          <Metadata>
            <Item Key="ProviderName">Twitter</Item>
            <Item Key="authorization_endpoint">https://api.twitter.com/oauth/authenticate</Item>
            <Item Key="access_token_endpoint">https://api.twitter.com/oauth/access_token</Item>
            <Item Key="request_token_endpoint">https://api.twitter.com/oauth/request_token</Item>
            <Item Key="ClaimsEndpoint">https://api.twitter.com/1.1/account/verify_credentials.json?include_email=true</Item>
            <Item Key="ClaimsResponseFormat">json</Item>
            <Item Key="client_id">Your Twitter application consumer key</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_TwitterSecret" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="user_id" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="screen_name" />
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="twitter.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Replace the value of **client_id** with the consumer key that you previously recorded.
5. Save the file.

### Upload the extension file for verification

By now, you have configured your policy so that Azure AD B2C knows how to communicate with your Twitter account. Try uploading the extension file of your policy just to confirm that it doesn't have any issues so far.

1. On the **Custom Policies** page in your Azure AD B2C tenant, select **Upload Policy**.
2. Enable **Overwrite the policy if it exists**, and then browse to and select the *TrustFrameworkExtensions.xml* file.
3. Click **Upload**.

## Register the claims provider

At this point, the identity provider has been set up, but it’s not available in any of the sign-up or sign-in screens. To make it available, you create a duplicate of an existing template user journey, and then modify it so that it also has the Twitter identity provider.

1. Open the *TrustFrameworkBase.xml* file from the starter pack.
2. Find and copy the entire contents of the **UserJourney** element that includes `Id="SignUpOrSignIn"`.
3. Open the *TrustFrameworkExtensions.xml* and find the **UserJourneys** element. If the element doesn't exist, add one.
4. Paste the entire content of the **UserJourney** element that you copied as a child of the **UserJourneys** element.
5. Rename the ID of the user journey. For example, `SignUpSignInTwitter`.

### Display the button

The **ClaimsProviderSelection** element is analogous to an identity provider button on a sign-up or sign-in screen. If you add a **ClaimsProviderSelection** element for a Twitter account, a new button shows up when a user lands on the page.

1. Find the **OrchestrationStep** element that includes `Order="1"` in the user journey that you created.
2. Under **ClaimsProviderSelects**, add the following element. Set the value of **TargetClaimsExchangeId** to an appropriate value, for example `TwitterExchange`:

    ```xml
    <ClaimsProviderSelection TargetClaimsExchangeId="TwitterExchange" />
    ```

### Link the button to an action

Now that you have a button in place, you need to link it to an action. The action, in this case, is for Azure AD B2C to communicate with a Twitter account to receive a token.

1. Find the **OrchestrationStep** that includes `Order="2"` in the user journey.
2. Add the following **ClaimsExchange** element making sure that you use the same value for the ID that you used for **TargetClaimsExchangeId**:

    ```xml
    <ClaimsExchange Id="TwitterExchange" TechnicalProfileReferenceId="Twitter-OAUTH1" />
    ```

    Update the value of **TechnicalProfileReferenceId** to the ID of the technical profile you created earlier. For example, `Twitter-OAUTH1`.

3. Save the *TrustFrameworkExtensions.xml* file and upload it again for verification.

::: zone-end

::: zone pivot="b2c-user-flow"

## Add Twitter identity provider to a user flow 

1. In your Azure AD B2C tenant, select **User flows**.
1. Click the user flow that you want to the Twitter identity provider.
1. Under the **Social identity providers**, select **Twitter**.
1. Select **Save**.
1. To test your policy, select **Run user flow**.
1. For **Application**, select the web application named *testapp1* that you previously registered. The **Reply URL** should show `https://jwt.ms`.
1. Click **Run user flow**

::: zone-end

::: zone pivot="b2c-custom-policy"

## Update and test the relying party file

Update the relying party (RP) file that initiates the user journey that you created.

1. Make a copy of *SignUpOrSignIn.xml* in your working directory, and rename it. For example, rename it to *SignUpSignInTwitter.xml*.
1. Open the new file and update the value of the **PolicyId** attribute for **TrustFrameworkPolicy** with a unique value. For example, `SignUpSignInTwitter`.
1. Update the value of **PublicPolicyUri** with the URI for the policy. For example,`http://contoso.com/B2C_1A_signup_signin_twitter`
1. Update the value of the **ReferenceId** attribute in **DefaultUserJourney** to match the ID of the new user journey that you created (SignUpSignTwitter).
1. Save your changes, upload the file, and then select the new policy in the list.
1. Make sure that Azure AD B2C application that you created is selected in the **Select application** field, and then test it by clicking **Run now**.

::: zone-end