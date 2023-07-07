# Usage of Sync API

In order to provide state synchronization between a user's devices we must add [Sync API](../core/sync/readme.md) support. As per Sync API requirements, the user [must sign an additional message](../core/sync/sync-protocol.md#generating-a-message-to-sign) to enable state synchronization. 

## Stores 

State synchronization is achieved by sharing the same key value stores among every client that the user
registered in Push API. For Push API there are the following stores:


### Push Subscription Store

#### Store Name
`com.walletconnect.notify.pushSubscription`

#### Store Key

Key of [Push Subscription Store](#push-subscription-store) must be equal to `topic` from
[`PushSubscription`](#pushsubscription) data structure. 

#### PushSubscription

```typescript
interface PushSubscription {
    topic: string;
    account: string;
    relay: RelayerTypes.ProtocolOptions;
    metadata: Metadata;
    scope: ScopeMap;
    expiry: number;
    symKey: string;
}
```

### Push Subscription Protocol in multiclient environment

#### Definitions
- W1 - clients that have access to W (Wallet) blockchain account keys.
- Wx - peer clients that have access to W (Wallet) blockchain account keys. 

#### Pre-requisities
The Push subscribe flow will require a dapp to host a static json file which will contain a DID document compliant with `did:web` method as specified [here](https://w3c-ccg.github.io/did-method-web/). In this DID document we will specify a X25519 public key that will be used by the Push API protocol to derive a symmetric key for the Push topic.

#### Protocol
Subscribe protocol will be established as follows:

1. W1 fetches public key X from the did:web document
2. Subscribe topic is derived from the sha256 hash of public key X
3. W1 generates key pair Y
4. W1 derives symmetric key S with keys X and Y
5. W1 sends push subscribe request (type 1 envelope) on subscribe topic with subscriptionAuth
7. Response topic is derived from the sha256 hash of symmetric key S
8. W1 subscribes to response topic
9. Cast Server receives push subscribe request on subscribe topic
10. Cast Server derives symmetric key S and decrypts subscriptionAuth
11. Cast Server triggers webhook to notify Dapp of new registered address
12. Cast Server generates key pair Z
13. Cast Server derives symmetric key P with keys Y and Z
14. Cast Server responds to push subscribe request on response topic
15. W1 receives push subscribe response on the response topic
16. W1 derives symmetric key P
17. Push topic is derived from the sha256 hash of the symmetric key P
18. W1 subscribes to push topic for future push messages
19. W1 syncs the push subscription with Wx
20. Wx stores the push subscription
21. Wx subscribe to push topic coming from the sync request.