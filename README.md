# diff-sync-js

A JavaScript implementation of Neil Fraser Differential Synchronization Algorithm

Diff Sync Writing: https://neil.fraser.name/writing/sync/

## Use Case

Differential synchronization algorithm keep two or more copies of the same document synchronized with each other in real-time. The algorithm offers scalability, fault-tolerance, and responsive collaborative editing across an unreliable network.

## Demo

http://wztechs.com/diff-sync-text-editor/demo/client/

## How to install

When use npm
`npm install diff-sync-js`

When use html
`<script src="./dist/diffSync.js"></script>`

## Dependencies

`json-fast-patch`

## To Test

`npm run test`

## How to use

1. Initial diffSync instance

```
    var diffSync = new DiffSyncAlghorithm({
        jsonpatch: jsonpatch,
        thisVersion: "m",
        senderVersion: "n",
        useBackup: true,
        debug: true
    });
```

2. Initialize container

```
    var container = {};
    diffSync.initObject(container, mainText);
```

3. When Send Payload

```
    diffSync.onSend({
        container,
        mainText,
        whenSend(senderVersion, edits) {
            send({
                type: "PATCH",
                {
                    senderVersion,
                    edits
                }
            });
        }
    });
```

4. When Receive Payload

```
    diffSync.onReceive({
        payload,
        container,
        onUpdateMain(patches, operations) {
            mainText = jsonpatch.applyPatch(mainText, operations).newDocument;
        },
        afterUpdate(senderVersion) {
            send({
                type: "ACK",
                payload: {
                    senderVersion
                }
            });
        }
    });
```

5. When Receive Ack

```
    diffSync.onAck(container, payload);
```

## API

constructor

```
     /**
     * @param {object} options.jsonpatch json-fast-patch library instance (REQUIRED)
     * @param {string} options.thisVersion version tag of the receiving end
     * @param {string} options.senderVersion version tag of the sending end
     * @param {boolean} options.useBackup indicate if use backup copy (DEFAULT true)
     * @param {boolean} options.debug indicate if print out debug message (DEFAULT false)
     */
    constructor({ jsonpatch, thisVersion, senderVersion, useBackup = true, debug = false })

```

initObject

```
    /**
     * Initialize the container
     * @param {object} container any
     * @param {object} mainText any
     */
    initObject(container, mainText)
```

onReceive

```
    /**
     * On Receive Packet
     * @param {object} options.payload payload object that contains {thisVersion, edits}. Edits should be a list of {senderVersion, patch}
     * @param {object} options.container container object {shadow, backup}
     * @param {func} options.onUpdateMain (patches, patchOperations, shadow[thisVersion]) => void
     * @param {func} options.afterUpdate (shadow[senderVersion]) => void
     * @param {func} options.onUpdateShadow (shadow, patch) => newShadowValue
     */
    onReceive({ payload, container, onUpdateMain, afterUpdate, onUpdateShadow })
```

onSend

```
    /**
     * On Sending Packet
     * @param {object} options.container container object {shadow, backup}
     * @param {object} options.mainText any
     * @param {func} options.whenSend (shadow[senderVersion], shadow.edits) => void
     * @param {func} options.whenUnchange (shadow[senderVersion]) => void
     */
    onSend({ container, mainText, whenSend, whenUnchange })

```

onAck

```
     /**
     * Acknowledge the other side when no change were made
     * @param {object} container container object {shadow, backup}
     * @param {object} payload payload object that contains {thisVersion, edits}. Edits should be a list of {senderVersion, patch}
     */
    onAck(container, payload)
```

onAck

```
    /**
     * Acknowledge the other side when no change were made
     * @param container container object {shadow, backup}
     * @param payload payload object that contains {thisVersion, edits}. Edits should be a list of {senderVersion, patch}
     */
    onAck(container, payload)
```

clearOldEdits

```
    /**
     * clear old edits
     * @param {object} shadow
     * @param {string} version
     */
    clearOldEdits(shadow, version)
```

strPatch

```
    /**
     * apply patch to string
     * @param {string} val
     * @param {patch} patch
     * @return string
     */
    strPatch(val, patch)
```


# Q&A SYT-Theorie 20.11.2024

- Which implementation of diff-patch-Alg?
    - GUARANTEED DELIVERY METHOD
- Where are the documents & the shadows?
    - <img width="784" alt="image" src="https://github.com/user-attachments/assets/b8da2e8b-a1cb-435e-ae3b-edf70654bb2e">
- How and Why can we adjust the sync-cycle? What are the dis-advantages?
    - Implement an adaptive synchronization system that modifies the sync frequency dynamically based on activity levels.
    - Potential Advantages:
        - Better Efficiency
        - Resource Management
    - Potential Disadvantages
        - Lost Data
        - Collisions
        - Unnecessary Complexity
- Where/How is the edit-Stack implemented?
 ````js 
 if (mainText !== null && mainText !== undefined) {
            container.shadow = {
                [thisVersion]: 0,
                [senderVersion]: 0,
                value: jsonpatch.deepClone(mainText),
                edits: []
};
````
  - The **edits** represent the edit-stack in form of an array. 
- Is it possible to deploy a Peer-to-Peer Version of Mr Wei's impementation?
  - In the current state, it only supports the Client-Server communication. To change it into a Peer-to-Peer Version, each peer has to be able to send and receive edits. If receiving the an update, a peer has to apply the patch on his shadow and update his version. 
- How is it possible to use the API in other SS-Projects?
- Are the JSON-Docs interchangable with other kind of Docs?
- How is Mr Wei solving the conflicts?
