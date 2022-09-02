# EPNS-SDK starter kit

This starter-kit is meant to showcase developers on how to use the EPNS SDK packages - 

* [@epnsproject/sdk-restapi](https://www.npmjs.com/package/@epnsproject/sdk-restapi) Provides access to EPNS backend APIs.
* [@epnsproject/sdk-uiweb](https://www.npmjs.com/package/@epnsproject/sdk-uiweb) Provides React based components to show Notifications, Spam, SubscribedModal etc for dApps.
* [@epnsproject/sdk-uiembed](https://www.npmjs.com/package/@epnsproject/sdk-uiembed) Provides vanilla JS sidebar notifications for any  dApp.

### CRA-Typescript
This particular kit is built out using CRA, Typescript. The SDK packages should also work out for React using plain JS.

## Getting started
```bash
git clone https://github.com/ethereum-push-notification-service/epns-sdk-starter-kit.git
```

```bash
cd epns-sdk-starter-kit
```

```bash
yarn install
```
```bash
yarn start
```

## Dependencies
If your are trying to build out a separate dApp following this starter-kit example, some of the following dependencies are needed for the SDK and any dApp to work.

1. `@epnsproject/sdk-uiweb` has a `peerDependency` on `styled-components`

```bash
yarn add styled-components
```

2. Since its a dApp, the following are the **web3** dependencies that you can install for wallet connection
```bash
 yarn add ethers
```

3. Needed only if you are using [web3-react](https://github.com/Uniswap/web3-react). You are free to use any other React based web3 solution.
```bash
yarn add @web3-react/core @web3-react/injected-connector
```

**But no need to install these if you are using the `starter-kit` itself since we have already installed these for you so that you can focus on how to use the EPNS-SDK packages**

## App walkthrough

The App has following features-

| Page    | Features    | SDK package used |
|----------|---------|---------|
| Notifications    | notifications, <br/>spams, <br/>subscribed modal  |  @epnsproject/sdk-uiweb, <br/>@epnsproject/sdk-restapi    |
| Channels     | get channel details for a specific channel, <br/>search for channel(s), <br/>get channel subscribers, <br/>is the logged-in user subscribed to the channel, <br/>opt in a channel, <br/>opt out a channel  | @epnsproject/sdk-restapi      |
| Payloads     | send notification for different use cases  | @epnsproject/sdk-restapi      |
| Embed | sidebar notifications for the logged in user if subscribed on EPNS  |   @epnsproject/sdk-uiembed    |

**We have extracted some snippets from the actual source code of the `starter-kit` files mentioned below to give you a snapshot view of what all SDK features are used in this dApp. But to make sure you are following along correctly please refer to the source code itself in the files mentioned.**

**Also the detailed SDK docs are hyperlinked in the feature's header itself**


*If you have got the wallet connection logic down, you can start referring from section [3](#3-features-usage) onwards.*

### 1. Connecting to the Wallet
Any dApp will require a wallet connection logic before it is usable. We have handled that for you in `App.tsx`. If you want to tinker around  with that, check the below component.

```typescript
import ConnectButton from './components/connect';
```

### 2. Wallet connection Props to be used throughout the dApp
We basically derive the account, signer and some other wallet connection properties to use throughout the dApp with the SDK.

```typescript
const { chainId, account, active, error, library  } = useWeb3React();
```
We store this data in the web3Context and make it available across the dApp for later use.

### 3. Features usage


NOTIFICATIONS PAGE (`src/pages/notifications/index.tsx`)
```typescript
import * as EpnsAPI from "@epnsproject/sdk-restapi";
import { NotificationItem, chainNameType, SubscribedModal } from '@epnsproject/sdk-uiweb';
```

#### [Fetching Notifications](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#fetching-user-notifications)

```typescript
const notifications = await EpnsAPI.user.getFeeds({
  user: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // user address in CAIP
  env: 'staging'
});
```

#### [Displaying Notifications](https://github.com/ethereum-push-notification-service/epns-sdk/tree/main/packages/uiweb#notification-item-component)

```typescript
{notifications.map((oneNotification, i) => {
    const { 
    cta,
    title,
    message,
    app,
    icon,
    image,
    url,
    blockchain,
    secret,
    notification
    } = oneNotification;

    return (
      <NotificationItem
        key={`notif-${i}`}
        notificationTitle={secret ? notification['title'] : title}
        notificationBody={secret ? notification['body'] : message}
        cta={cta}
        app={app}
        icon={icon}
        image={image}
        url={url}
        theme={theme}
        chainName={blockchain as chainNameType}
      />
    );
})}
```

#### [Fetching Spams](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#fetching-user-spam-notifications)

```typescript
const spams = await EpnsAPI.user.getFeeds({
  user: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // user address in CAIP
  spam: true,
  env: 'staging'
});
```

#### [Displaying Spams](https://github.com/ethereum-push-notification-service/epns-sdk/tree/main/packages/uiweb#notification-item-component)

```typescript
{spams ? (
    <NotificationListContainer>
        {spams.map((oneNotification, i) => {
        const { 
        cta,
        title,
        message,
        app,
        icon,
        image,
        url,
        blockchain,
        secret,
        notification
        } = oneNotification;

        return (
            <NotificationItem
                key={`spam-${i}`}
                notificationTitle={secret ? notification['title'] : title}
                notificationBody={secret ? notification['body'] : message}
                cta={cta}
                app={app}
                icon={icon}
                image={image}
                url={url}
                theme={theme}
                chainName={blockchain as chainNameType}
                // optional parameters for rendering spambox
                isSpam
                subscribeFn={subscribeFn} // see below
                isSubscribedFn={isSubscribedFn} // see below
            />
        );
    })}
```


```typescript
const subscribeFn = async () => {
  // opt in to the spam notification item channel
}
```
we can use this `@epnsproject/sdk-restapi` method to do that - [subscribe](https://github.com/ethereum-push-notification-service/epns-sdk/tree/main/packages/restapi/README.md#opt-in-to-a-channel)


```typescript
const isSubscribedFn = async () => {
  // return boolean which says whether the channel for the 
  // spam notification item is subscribed or not by the user.
}
```
we can use this `@epnsproject/sdk-restapi` method to find out that - [getSubscriptions](https://github.com/ethereum-push-notification-service/epns-sdk/tree/main/packages/restapi/README.md#fetching-user-subscriptions)



#### Parsing raw [Feeds API data](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#fetching-user-notifications) using [utils](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#parsing-notifications) method `parseApiResponse`
Utils method to parse raw EPNS Feeds API response into a pre-defined shape as below.
```typescript
// fetch some raw feeds data
const apiResponse = await EpnsAPI.user.getFeeds({
  user: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // user address
  raw: true,
  env: 'staging'
});
// parse it to get a specific shape of object.
const parsedResults = EpnsAPI.utils.parseApiResponse(apiResponse);

const [oneNotification] = parsedResults;

// Now this object can be directly used by for e.g. "@epnsproject/sdk-uiweb"  NotificationItem component as props.

const {
  cta,
  title,
  message,
  app,
  icon,
  image,
  url,
  blockchain,
  secret,
  notification
} = oneNotification;

```
*We get the above `keys` after the parsing of the API repsonse.*

#### SubscribedModal

```typescript
    const [showSubscribe, setShowSubscribe] = useState(false);

    const toggleSubscribedModal = () => {
        setShowSubscribe((lastVal) => !lastVal);
    };


    // JSX
    {showSubscribe ? <SubscribedModal onClose={toggleSubscribedModal}/> : null}
```

CHANNELS PAGE (`src/pages/channels/index.tsx`)

```typescript
import * as EpnsAPI from '@epnsproject/sdk-restapi';
```
#### [Fetch Channel Data](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#fetching-channel-details)

```typescript
const channelData = await EpnsAPI.channels.getChannel({
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // channel address in CAIP
  env: 'staging'
});
```

#### [Searching for channel(s)](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#searching-for-channels)
```typescript
const channelsData = await EpnsAPI.channels.search({
  query: 'epns', // a search query
  page: 1, // page index
  limit: 20, // no of items per page
  env: 'staging'
});
```

#### [DEPRECATED-Fetch Channel Subscribers](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#deprecated)

```typescript
const subscribers = await EpnsAPI.channels._getSubscribers({
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // channel address in CAIP
  env: 'staging'
});
```

#### [Fetch User subscriptions](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#fetching-user-subscriptions)

```typescript
const subscriptions = await EpnsAPI.user.getSubscriptions({
  user: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // user address in CAIP
  env: 'staging'
});
```

#### [Opt-In to a channel](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#opt-in-to-a-channel) 

```typescript
const _signer = library.getSigner(account); // from useWeb3()
 //
 //
 //
await EpnsAPI.channels.subscribe({
  signer: _signer,
  channelAddress: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // channel address in CAIP
  userAddress: 'eip155:42:0x52f856A160733A860ae7DC98DC71061bE33A28b3', // user address in CAIP
  onSuccess: () => {
   console.log('opt in success');
  },
  onError: () => {
    console.error('opt in error');
  },
  env: 'staging'
})
```

#### [Opt-Out of a channel](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#opt-out-to-a-channel) 

```typescript
const _signer = library.getSigner(account); // from useWeb3()
 //
 //
 //
await EpnsAPI.channels.unsubscribe({
  signer: _signer,
  channelAddress: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // channel address in CAIP
  userAddress: 'eip155:42:0x52f856A160733A860ae7DC98DC71061bE33A28b3', // user address in CAIP
  onSuccess: () => {
   console.log('opt out success');
  },
  onError: () => {
    console.error('opt out error');
  },
  env: 'staging'
})
```

PAYLOADS PAGE (`src/pages/payloads/index.tsx`)

#### [Send Notifications](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/restapi/README.md#sending-notification)

##### **direct payload for single recipient(target)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 3, // target
  identityType: 2, // direct payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  recipients: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // recipient address
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **direct payload for group of recipients(subset)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 4, // subset
  identityType: 2, // direct payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  recipients: ['eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', 'eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1'], // recipients addresses
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **direct payload for all recipients(broadcast)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 1, // broadcast
  identityType: 2, // direct payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **IPFS payload for single recipient(target)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 3, // target
  identityType: 1, // ipfs payload
  ipfsHash: 'bafkreicuttr5gpbyzyn6cyapxctlr7dk2g6fnydqxy6lps424mcjcn73we', // IPFS hash of the payload
  recipients: 'eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1', // recipient address
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **IPFS payload for group of recipients(subset)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 4, // subset
  identityType: 1, // ipfs payload
  ipfsHash: 'bafkreicuttr5gpbyzyn6cyapxctlr7dk2g6fnydqxy6lps424mcjcn73we', // IPFS hash of the payload
  recipients: ['eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1', 'eip155:42:0x52f856A160733A860ae7DC98DC71061bE33A28b3'], // recipients addresses
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **IPFS payload for all recipients(broadcast)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 1, // broadcast
  identityType: 1, // direct payload
  ipfsHash: 'bafkreicuttr5gpbyzyn6cyapxctlr7dk2g6fnydqxy6lps424mcjcn73we', // IPFS hash of the payload
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **minimal payload for single recipient(target)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 3, // target
  identityType: 0, // Minimal payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  recipients: 'eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1', // recipient address
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **minimal payload for a group of recipient(subset)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 4, // subset
  identityType: 0, // Minimal payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  recipients: ['eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1', 'eip155:42:0x52f856A160733A860ae7DC98DC71061bE33A28b3'], // recipients address
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **minimal payload for all recipients(broadcast)**
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 1, // broadcast
  identityType: 0, // Minimal payload
  notification: {
    title: `[SDK-TEST] notification TITLE:`,
    body: `[sdk-test] notification BODY`
  },
  payload: {
    title: `[sdk-test] payload title`,
    body: `sample msg body`,
    cta: '',
    img: ''
  },
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **graph payload for single recipient(target)**
***Make sure the channel has the graph id you are providing!!***
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 3, // target
  identityType: 3, // Subgraph payload
  graph: {
    id: '_your_graph_id',
    counter: 3
  },
  recipients: 'eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1', // recipient address
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **graph payload for group of recipients(subset)**
***Make sure the channel has the graph id you are providing!!***
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 4, // subset
  identityType: 3, // graph payload
  graph: {
    id: '_your_graph_id',
    counter: 3
  },
  recipients: ['eip155:42:0xCdBE6D076e05c5875D90fa35cc85694E1EAFBBd1', 'eip155:42:0x52f856A160733A860ae7DC98DC71061bE33A28b3'], // recipients addresses
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```

##### **graph payload for all recipients(broadcast)**
***Make sure the channel has the graph id you are providing!!***
```typescript
// apiResponse?.status === 204, if sent successfully!
const apiResponse = await EpnsAPI.payloads.sendNotification({
  signer,
  type: 1, // broadcast
  identityType: 3, // graph payload
  graph: {
    id: '_your_graph_id',
    counter: 3
  },
  channel: 'eip155:42:0xD8634C39BBFd4033c0d3289C4515275102423681', // your channel address
  env: 'staging'
});
```




EMBED PAGE (`src/pages/embed/index.tsx`)
#### [Embed - Sidebaar notifications](https://github.com/ethereum-push-notification-service/epns-sdk/blob/main/packages/uiembed/README.md#uiembed) 


```typescript
import { useEffect, useContext } from 'react';
import { EmbedSDK } from "@epnsproject/sdk-uiembed";
import Web3Context from '../../context/web3Context';

const EmbedPage = () => {
    const { account, chainId } = useContext<any>(Web3Context);

    useEffect(() => {
        if (account) { // 'your connected wallet address'
          EmbedSDK.init({
            chainId,
            headerText: 'Hello Hacker Dashboard', // optional
            targetID: 'sdk-trigger-id', // mandatory
            appName: 'hackerApp', // mandatory
            user: account, // mandatory
            viewOptions: {
                type: 'sidebar', // optional [default: 'sidebar', 'modal']
                showUnreadIndicator: true, // optional
                unreadIndicatorColor: '#cc1919',
                unreadIndicatorPosition: 'top-right',
            },
            theme: 'light',
            onOpen: () => {
              console.log('-> client dApp onOpen callback');
            },
            onClose: () => {
              console.log('-> client dApp onClose callback');
            }
          });
        }
    
        return () => {
          EmbedSDK.cleanup();
        };
      }, [account, chainId]);


    return (
        <div>
          <h2>Embed Test page</h2>


          <button id="sdk-trigger-id">trigger button</button>
        </div>
    );
}
```