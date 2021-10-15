# Monetizr Challenges SDK for Unity

This SDK is used to communicate with Monetizr challenges API.

# Installation

Clone the repository and import `monetizr_challenges_unity_sdk_1.0.unitypackage` into your project using `Assets/Import Package/Custom Package...`.

# Usage

You have to create and use one instance of `ChallengesClient`  class and use it to communicate with the Monetizr Challenges API. You also have to set the current user's information in `ChallengesClient.playerInfo` (at least the identifier) to use the SDK. Challenge list can be retrieved using `GetList`. Whenever a progress is being made by player, `UpdateChallenge` has to be called. When the challenge is done, a claim button must be shown in the challenge menu and `ClaimChallenge` must be called whenever player clicks it.

# Reference
[Detailed Documentation](#documentation)
## Monetizr.Challenges namespace
## ChallengesClient

`ChallengesClient(string apiKey, int timeout = 30)` – creates an instance of ChallengesClient and sets the API key used by the SDK to apiKey.<br>
`GetList()` – gets a list of challenges available to player.<br>
`GetSingle(string id)` – gets a single challenges identified by id.<br>
`UpdateStatus(Challenge challenge, int progress, Action onSuccess, Action onFailure)` – updates the player's progress.<br>
`ClaimChallenge(Challenge challenge, Action onSuccess, Action onFailure)` – must be called when player click the claim button on a completed challenge.<br>
`playerInfo` – used to get and set player's information.<br>

## AssetsHelper

`Download2DAsset(Challenge.Asset asset, Action<Challenge.Asset, Sprite> onAssetDownloaded)` – downloads a 2D asset and returns it as a Sprite in onAssetDownloaded callback.

`DownloadAssetData(Challenge.Asset asset, Action<Challenge.Asset, byte[]> onAssetDownloaded)` – Downloads any type of asset and returns its data as an array of bytes in onAssetDownloadedCallback.

## PlayerInfo

`location` – location of the current user (currently not used).<br>
`age` – age of the current user (optional).<br>
`gameType` – type of the game (currently not used).<br>
`playerId` – a unique identifier of the user (mandatory).<br>

## Challenge
`id` – identification number of the challenge (must be passed to UpdateChallenge and ClaimChallenge)
`title` – title of the challenge (must be visible in the challenge menu).<br>
`content` – description of the challenge (must be visible in the challenge menu).<br>
`progress` – players progress (must be visible in the challenge menu).<br>
`reward` – type of reward when the challenge is done (0 for in-game money).<br>
`assets` – a list of assets of the challenge (see Challenge.Asset).<br>

## Challenge.Asset
`id` – identification number of the asset (can be ignored).<br>
`type` – type of the asset (`icon` or `banner`).<br>
`title` – name of the asset (can be ignored).<br>
`url` – the location of the asset.<br>

---

## Sample Code

To test if everything works, create a script with the following code and attach it to an object in your scene. If Challenges are set up correctly, you will see a list of challenges appear on that object in the inspector after the game starts.

```csharp
using System.Collections.Generic;
using UnityEngine;
using Monetizr.Challenges;

public class MonetizrChallengesTest : MonoBehaviour
{
    [SerializeField]
    List<Challenge> challenges;

    private ChallengesClient challengesClient;

    private void Awake()
    {
        challengesClient = new ChallengesClient("your_api_key")
        {
            playerInfo = new PlayerInfo("City", 18, "action", "user")
        };

        UpdateChallengesList();
    }

    private async void UpdateChallengesList()
    {
        challenges = await challengesClient.GetList();
    }
}
```

# Documentation

## ChallengesClient
There should only be one instance of `ChallengesClient` in your project.<br> 
Remember to set the value of `ChallengesClient.playerInfo`.
```csharp
challengesClient = new ChallengesClient("your_api_key")
{
    playerInfo = new PlayerInfo("City", 18, "action", "user")
};
```

### **GetList()**
```csharp
async Task<List<Challenge>> GetList()
```
Use `GetList()` to retrieve the list of all challenges available to the player.<br>
If retrieving challenges from the server fails, an **empty List** will be returned.<br>

### **GetSingle()**
```csharp
async Task<Challenge> GetSingle(string id)
```
Use `GetSingle()` to retrieve a single challenge with a matching ID.<br> 
If retrieving the challenge from the server fails **null** will be returned.

### **UpdateStatus()**
```csharp
async Task UpdateStatus(Challenge challenge, int progress, Action onSuccess = null, Action onFailure = null)
```
Use `UpdateStatus()` to update progress of the given challenge.<br> 
Progress gets clamped to a range between 0 and 100.<br> 
After status update attempt, depending on server response either **onSuccess** or **onFailure** will be invoked.

### **Claim()**
```csharp
async Task Claim(Challenge challenge, Action onSuccess = null, Action onFailure = null)
```
Use `Claim()` to mark the challenge as claimed by the player.<br>
After claim attempt, depending on server response either **onSuccess** or **onFailure** will be invoked.

---

## AssetsHelper
`AssetsHelper` is a static class that lets you download assets associated with challenges. 
```csharp
StartCoroutine(AssetsHelper.Download2DAsset(challenges[0].assets[0], (asset, sprite) => { image.sprite = sprite; }, () => Debug.Log("Asset download failed")));
```

### **Download2DAsset()**
```csharp
static IEnumerator Download2DAsset(Challenge.Asset asset, Action<Challenge.Asset, Sprite> onAssetDownloaded, Action onDownloadFailed = null)
```
Use `Download2DAsset()` to download assets of type `icon` or `banner`.<br>
If asset is successfuly downloaded, a sprite will be generated and returned in **onAssetDownloaded**.<br>
If asset download fails **onDownloadFailed** will be invoked.

### **DownloadAssetData()**
```csharp
static IEnumerator DownloadAssetData(Challenge.Asset asset, Action<Challenge.Asset, byte[]> onAssetDownloaded, Action onDownloadFailed = null)
```
Use `DownloadAssetData()` to download any type of an asset as an array of bytes. This allows for custom handling of 2D, 3D or any other type of asset types.<br>
If asset is successfuly downloaded, an array of bytes will be returned in **onAssetDownloaded**.<br>
If asset download fails **onDownloadFailed** will be invoked.
