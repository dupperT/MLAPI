---
title: Connection Approval
permalink: /wiki/connection-approval/
---

During every new connection the MLAPI performs a handshake on top of the one(s) done by the transport. This is to ensure that the NetworkConfig's match up between the Client and Server. In the NetworkConfig you can specify to enable ConnectionApproval. Connection approval will let you decide on a per connection basis if the connection should be allowed.

However, when ConnectionApproval is true you are also required to provide a callback where you put your approval logic inside. (Server only) Example:
```csharp
private void Setup() 
{
    NetworkingManager.Singleton.ConnectionApprovalCallback = ApprovalCheck;
    NetworkingManager.Singleton.StartHost();
}

private void ApprovalCheck(byte[] connectionData, ulong clientId, MLAPI.NetworkingManager.ConnectionApprovedDelegate callback)
{
    //Your logic here
    bool approve = true; //If approve is true, the connection gets added. If it's false. The client gets disconnected

    ulong prefabHash = SpawnManager.GetPrefabHashFromGenerator("Object-hash-string-here"); // The prefab hash of the gameobject you want to spawn. Use null to use the default player prefab
    
    bool createPlayerObject = true; //Create player object. If True it spawns the prefabHash, if false it doesn't.
    
    callback(createPlayerObject, prefabHash, approve, positionToSpawnAt, rotationToSpawnWith);
}
```
### Setting up the Player
When setting up a prefab you must add the Networked Object script to your gameobject. This will give you a string field to set the Hash. 

![Video showing the install process](https://i.imgur.com/TLV1p9j.png)

You must also register the prefab in the Networked Prefabs section under the Networking Manager Script. 

![Video showing the install process](https://i.imgur.com/yOfBRyL.png)

If you tick 'Default Player prefab' you can now change the 'Object-hash-string-here' to your Player object hash.

Before:

    ulong prefabHash = SpawnManager.GetPrefabHashFromGenerator("Object-hash-string-here"); // The prefab hash. Use null to use the default player prefab
    
After:

    ulong prefabHash = SpawnManager.GetPrefabHashFromGenerator("Test"); // The prefab hash. Use null to use the default player prefab


### Connection data
The connectionData parameter is any custom data of your choice that the client should send to the server. Usually, this should be some sort of ticket, room password or similar that will decide if a connection should be approved or not. The connectionData is specified on the Client side in the NetworkingConfig supplied when connecting. Example:
```csharp
NetworkingManager.Singleton.NetworkConfig.ConnectionData = System.Text.Encoding.ASCII.GetBytes("room password");
NetworkingManager.Singleton.StartClient();
```
The ConnectionData will then be passed to the server and it will decide if the client will be approved or not.


### Timeout
The MLAPI uses a callback system in order to allow for external validation. For example, you might have a steam authentication ticket sent as the ConnectionData (encrypted and authenticated by the MLAPI) that you want to validate against steams servers. This can take some time. If you don't call the callback method within the time specified in the ``ClientConnectionBufferTimeout`` configuration the connection will be dropped. This time starts counting when the transport has told the MLAPI about the connection. This means that you cannot attack the MLAPI by never sending the buffer, it will still time you out.


### Security
If connection approval is enabled. Any messages sent before a connection is setup are silently ignored.

#### Connection Data
If Encryption is enabled, the connection handshake with the buffer will be encrypted AND authenticated. (AES-256 encryption and HMAC-SHA-256 authentication). Please note that if the key exchange is not signed, a man in the middle attack can be done. If you plan on sending authentication tokens such as steam tickets. It's strongly suggested that you sign the handshake.
