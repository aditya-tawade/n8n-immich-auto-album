# 🚀 Auto-add recognized faces to a specific Immich Album (n8n)

Hi everyone,

I created an automation in **n8n** that checks for new photos of a specific person (using Immich's facial recognition) and automatically adds them to a specific Album.

This is perfect for automatically sharing photos of your kids, partner, or pets without manually selecting them every time.

### How it works:
1.  **Runs every night** (default: 03:00 AM).
2.  Resolves the **Person Name** to a **Person ID** automatically.
3.  Searches for all asset IDs linked to that Person ID.
4.  Adds them to your target **Album** (Immich automatically skips duplicates).

---

## 🛠️ Prerequisites & Finding IDs

Before importing, you need to gather a few things:

1.  **Immich API Key:**
    * Go to *Account Settings* -> *API Keys* -> *New API Key*.
2.  **The Album ID:**
    * Open the target album in Immich web.
    * Look at the URL: `.../albums/49302-abcd-1234`
    * The part after `/albums/` is your **Album ID**.
3.  **The Person Name:**
    * Exactly as it appears in Immich (e.g., "John Doe").

---

## 📦 Step 1: The Workflow JSON

Copy the code below and paste it directly into your n8n canvas (Ctrl+V / Cmd+V).

```json
{
  "name": "Immich - Auto Add Person to Album",
  "nodes": [
    {
      "parameters": {
        "jsCode": "// Extract the list of photos from the previous step\n// Note: Immich puts them in 'assets' -> 'items'\nconst photos = items[0].json.assets.items;\n\n// Map to a simple list of IDs\nconst ids = photos.map(photo => photo.id);\n\n// Return data to n8n\nreturn {\n  json: {\n    ids: ids\n  }\n};"
      },
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        1040,
        0
      ],
      "id": "c1164e7f-55aa-47a1-baf2-c637a219b7ad",
      "name": "Extract IDs",
      "executeOnce": true
    },
    {
      "parameters": {
        "method": "PUT",
        "url": "http://YOUR_IMMICH_IP:2283/api/albums/ENTER_ALBUM_ID_HERE/assets",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ { \"ids\": $json.ids } }}",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        1248,
        0
      ],
      "id": "74d38557-2fda-4b82-9934-95fa7c57d39e",
      "name": "Add to album"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "537d1414-e3c5-4bb9-a777-52264f151b5e",
              "name": "PersonName",
              "value": "ENTER_PERSON_NAME",
              "type": "string"
            },
            {
              "id": "dc54d126-4d91-4694-b7f8-88df7e02582b",
              "name": "AlbumID",
              "value": "ENTER_ALBUM_ID_HERE",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        416,
        0
      ],
      "id": "2b339c1c-37ab-454e-aa96-a366395f8e6a",
      "name": "Config: Name & Album"
    },
    {
      "parameters": {
        "url": "http://YOUR_IMMICH_IP:2283/api/search/person",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "name",
              "value": "={{ $json.PersonName }}"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        624,
        0
      ],
      "id": "ee2fda94-427e-463e-a143-77fc774773c9",
      "name": "Find Person ID"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=http://YOUR_IMMICH_IP:2283/api/search/metadata",
        "authentication": "genericCredentialType",
        "genericAuthType": "=httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={\n  \"personIds\": [\n    \"{{ $json.id }}\"\n  ],\n  \"page\": 1,\n  \"size\": 1000\n}",
        "options": {}
      },
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.3,
      "position": [
        832,
        0
      ],
      "id": "15268022-d0d1-4585-8952-1aaf52c95aa2",
      "name": "Fetch Assets"
    },
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "triggerAtHour": 3
            }
          ]
        }
      },
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.3,
      "position": [
        208,
        0
      ],
      "id": "acf71746-6e3b-43ea-bcb2-dff6b388a5de",
      "name": "Daily Trigger"
    }
  ],
  "pinData": {},
  "connections": {
    "Extract IDs": {
      "main": [
        [
          {
            "node": "Add to album",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Config: Name & Album": {
      "main": [
        [
          {
            "node": "Find Person ID",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Find Person ID": {
      "main": [
        [
          {
            "node": "Fetch Assets",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Fetch Assets": {
      "main": [
        [
          {
            "node": "Extract IDs",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Daily Trigger": {
      "main": [
        [
          {
            "node": "Config: Name & Album",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

---

## ⚙️ Step 2: Configuration

After importing the workflow, you need to configure a few nodes:

1.  **Set up Credentials:**
    * In n8n, go to **Credentials** -> **New** -> **Header Auth**.
    * Name: `Immich API`
    * Name (Header): `x-api-key`
    * Value: Paste your API Key.
    * **Important:** Apply this credential to all 3 HTTP Request nodes.

2.  **Edit Node: "Config: Name & Album"**
    * Replace `ENTER_PERSON_NAME` with the exact name.
    * Replace `ENTER_ALBUM_ID_HERE` with your Album UUID.

3.  **Edit HTTP Nodes:**
    * Replace `http://YOUR_IMMICH_IP:2283` with your actual server IP/Domain in all 3 HTTP nodes.
    * **Crucial:** In the final node (**Add to album**), you must **manually paste your Album ID into the URL field again** to replace `ENTER_ALBUM_ID_HERE`.

---

## 💡 First Run: Importing your entire history

By default, Immich only fetches **1,000 photos** at a time to save resources. If the person has, for example, 5,000 photos, the first run will only add the first 1,000.

**To import your full backlog:**
1.  Open the node **"Fetch Assets"**.
2.  Look at the JSON Body at the bottom.
3.  Change `"page": 1` to `"page": 2`.
4.  Run the workflow manually.
5.  Repeat this (Page 3, Page 4, etc.) until no new photos are added to the album.
6.  **Important:** When finished, set it back to `"page": 1`. This ensures the daily schedule always checks for the newest photos.

Enjoy!
