# Prepaid Card Instruments(PPI) integration

A program manager for a card program works with multiple entities to ensure the program runs smoothly. These entities may include banks, card networks, card fulfillment vendors, and other partners. With the help of [M2P](https://m2pfintech.com/), [LivQuik](https://livquik.com/) and [Syntizen](https://syntizen.com/) I built a Multi-tenant backend system that will allow my client to enable Prepaid cards to customers seamlessly.

### Understanding the space

#### What are Co-Branded Prepaid Cards?

Unlike a debit/credit card a Prepaid card is not linked to any bank account. Similar to a wallet, it can be filled with funds and can be used like a regular card.&#x20;

A co-branded card is a payment credential created in partnership with a company tied to driving consumer engagement and loyalty. Co-branded Visa cards enable the partners to build customer loyalty through strong card value propositions that incentivise their customers to pay with the card. With Visa co-branded cards, cardholders benefit from unique rewards on everyday purchases and access to a variety of benefits, sellers benefit from increased sales and customer loyalty, and issuers benefit from data and revenue generated.

#### VISA and M2P Partnership

[VISA](https://www.visa.co.in/) launched the 'Visa Ready to Launch' (VRTL) program in partnership with selected payment enablers such as M2PFintech(the enabler), Asia's largest API infrastructure company. VRTL is a plug-and-play, end-to-end card issuance platform that bundles products and services, enabling fintechs to swiftly launch new card programs and payment solutions.

My client, lets call it Company X. Company X is no-code credit lending solution. They wanted to add PPI in their suite of products that they offer to their tenants(T). These tenants are B2C companies that can seamlessly integrate with Company X to deliver PPI cards for its customers.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

The reason this hierarchy exists is majorly for 2 reasons.&#x20;

1. Finance is a heavily regulated space. A card program need to be PCI DSS compliant and VAPT cleared to go live. The further you go up the chain, the more the compliance gets complicated. Which in turn affects Go-to-market(GTM). Hence growing product companies look for the vendor with whom the integration can be done the quickest.
2. The lower you go on the chain, the more of it is abstracted which enables companies to integrate with minimal hassle.

#### Project Components

1.  **KYC:** Onboarding a customer requires doing KYC. There were two options of KYC.

    1. minKYC: All you needed was the Aadhard card and the OTP
    2. fullKYC: a.k.a videoKYC. The user has to join a video call with an agent and answer a bunch of questions. Enabled by Syntizen.

    fullKYC was more resilient than minKYC, hence the PPI card limit was higher for fullKYC users.
2. **Card Management:** It had APIs to view balance, transactions and card controls. It also includes the necessary APIs for the Card Program to debit, transfer funds from one account to another.
3. **Additional Card services:** This module included APIs for blocking, replacing and placing an order for a new card.
4. **Card Transaction Webhook:** For every card transaction performed by the card owner, webhooks will be sent by M2P. This needed to be stored for record purposes.

#### Under the hood

Let's get a little more deeper on where the funds are stored and how its moved around. VISA is the payment protocol, it has standard guidelines upon which M2P has built its infra. This also is a multi-tenant system indeed.&#x20;

After KYC verification, each user will be assigned a card entity. They are also allocated a virtual account inside M2P's Nodal account(powered by LivQuik). The card program maintains a wallet within LivQuik, that is used to credit funds to the customer's wallet whenever the **Load Funds API** is used. This needs to be facilitated by the card program through a Payment Gateway.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Later, there was also an easier way enabled by M2P. Since each customer maintains a virtual account at LivQuik Bank. They were able to generate UPI IDs for each of the wallet account. And funds can be directly transferred to this UPI ID. This was preferred as they above requires some Txn charges which needs to be beared by Company X.

#### Is there any money in it?

This is the percentage split for Company X.&#x20;

VISA(4%) -> M2P (X%) -> Company X (?)-> Tenants

Company X & M2P agree on a deal that they would get X% if company X is able to deliver a certain amount of transaction volume.  I am not sure if Company X has to pay a monthly fees to M2P for the infrastrcture.

And for the tenants, I have no clue again if there would be anything left to be given.

But Co-branded cards are built for the sole purpose of enabling companies to build loyalty programs and making transactions within their platform easier. It might also help them reduce PG costs if the transaction were to be done otherwise. What companies were most interested on is customer's transaction history. Since its a VISA card, the customer can use it for their common purchases. And the Tenants would have access to the complete transaction history. Not just the transactions on their platform.

### Tech Architecture

I built 2 services, an API service and a Webhook Handler. The services were written in Node.JS with Express. I used DynamoDB as our primary database. We diligently designed the Primary Key and Sort Key of the tables so all of our access patterns were handled. I containerised them and deployed them on the cheapest EC2 instance.&#x20;

* **API Service:** It was a fairly straightforward API wrapper. M2P mandated E2E encryption on all API requests. So I added Axios interceptors to encrypt and decrypt the request payloads and responses according to their guidelines. I used NGINX as a reverse proxy to route traffic to the backend server. The plan was to replicate API containers as per scale and load balance it via NGINX.
* **Webhook Handler:** The SLA for the handler was fairly large. And there was no way we are missing even a single webhook notification. So I configured a Lambda that consumes the webhook messages and pushes it into an SQS. The webhook handler polls the SQS and processes each message. I wrote a more elaborate article on it. [Link](https://dev.to/harishteens/i-built-a-strange-webhook-handler-in-nodejs-43b1) to the article.

#### Tenant Onboarding

I leveraged the native support for partioning in DynamoDB. The primary key was tenant ID. There was just 2 tables. One for users, and another for Transactions.&#x20;

To onboard a tenant, Company X simply had to generate them a unique Tenant ID and their corresponding API-Key using the in-house hashing function. The API-Key needs to be passed on every request. It ensured that its a valid tenant and also the request is accessing a user allocated in the tenant's namespace.

#### Observability

This was my first time setting up Observability dashboard on Grafana. I used OpenTSDB to store metrics and built a wholesome dashboard that was most important for us.

We needed 3 metrics,

1. Latency
2. Response Statuses
3. Request per second

I added interceptors everywhere to populate request TATs. Using the data, I was able to write queries on Grafana to build this below dashbaord.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption><p>Trace of a request</p></figcaption></figure>

More details on the linkedIn post below

{% embed url="https://www.linkedin.com/posts/harishteens_agency-services-backend-activity-7071844405892055040-3C0z?utm_source=share&utm_medium=member_desktop" %}

#### Conclusion

This was one of my first large scale project and I am fond of it. It took me about 45 days to build everything and get it to production. One of the founders by mistake sent me an email that contained how much they charged as a One Time Fee. It was 5x of what we were paid. And that is just for one client. Nonetheless, this project was a monumental one in my career. I am grateful that I got to work on this, Thanks to Company X.

