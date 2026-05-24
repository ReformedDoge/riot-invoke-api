### RiotInvoke Native API Reference
 [![Maintainer](https://img.shields.io/badge/maintained%20by-@ReformedDoge-gold.svg?style=flat-square)](https://github.com/ReformedDoge)

This document catalogs the native C++ Inter-Process Communication (IPC) endpoints exposed to the League of Legends CEF process via the `window.riotInvoke` bridge.

## Documented Namespaces

This repository documents endpoints spanning the following core client subsystems:
* **`Window.*`**: Native OS window controls (show, hide, minimize, restore, flash, resize, move, center, frame boundaries).
* **`Mouse.*`**: Drag-bar configurations and sizing constraints.
* **`File.*`**: Native OS dialog prompts for directories and saving configurations.
* **`RiotClient.*`**: Native clipboard access (copy, paste) and client termination.
## How to use `window.riotInvoke`

To call these endpoints directly from JavaScript, serialize your request into a JSON string and pass parameters inside the `params` array. Depending on the type of endpoint, you must parse the response differently:

### Case A: Standard Actions (No Return Data)
Endpoints that perform an action (like showing a window, copying text, or triggering a flash) do not return a payload. You do not need to parse the response.
```javascript
window.riotInvoke({
    request: JSON.stringify({
        name: "Window.Show",
        params: [] // "main" Passed as empty = current window targeted uxId
    })
});
```

### Case B: Plain Strings / Paths (Direct String Access)
Endpoints that prompt for files or directories return the path as a raw, unquoted string inside `envelope.result`. **Do not** call `JSON.parse` on `envelope.result`, or the JavaScript parser will throw a `SyntaxError` on Windows backslashes:
```javascript
window.riotInvoke({
    request: JSON.stringify({
        name: "File.RequestDirectoryPath",
        params: ["C:\\Program Files", "Select Folder", "Select"]
    }),
    onSuccess: function(response) {
        const envelope = JSON.parse(response);
        const dirPath = envelope.result; // Access directly as a raw string!
        console.log("Selected Folder: ", dirPath);
    }
});
```

### Case C: Structured Objects (Double JSON Parsing)
For endpoints that return structured objects (like `Window.ScreenData` or `Window.GetValidWindowSizes`), the C++ backend returns a nested string inside the response envelope. You must parse the envelope twice to access the payload:
```javascript
window.riotInvoke({
    request: JSON.stringify({
        name: "Window.ScreenData",
        params: ["", ""] // ["persistent_query_id"]
    }),
    onSuccess: function(response) {
        // 1. Parse the outer envelope returned by RiotInvoke
        const envelope = JSON.parse(response);
        
        // 2. Parse the actual stringified C++ payload packed inside the 'result' key
        const data = JSON.parse(envelope.result);
        
        console.log("ScreenData: ", data);
    }
});
```

---

## 🗂️ Table of Contents
1. [🖥️ Window Management](#1-window-management-window)
2. [🖱️ Mouse & Interaction](#2-mouse--interaction-mouse)
3. [📁 File System Prompts](#3-file-system-prompts-file)
4. [⚙️ Riot Client & System](#4-riot-client--system-riotclient)

---

## 1. Window Management (`Window.*`)
**Native Module:** `RiotInvoke_WindowModuleInit`

### `Window.Activate`
> Activates the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window (e.g., `"main"`) |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.Activate",
          params: []
      })
  });
  ```

---

### `Window.AddToTaskbar`
> Adds the ux window to the taskbar.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.AddToTaskbar",
          params: []
      })
  });
  ```

---

### `Window.ApplyMask`
> Applys the mask in the given image file path to the window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `maskPath` | String | path to the image file that is the window mask (must be absolute) |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.ApplyMask",
          params: [
              
              "C:\\test_mask.png" // Escaped absolute path to a transparent PNG
          ]
      }),
      onSuccess: function(res) {
          const envelope = JSON.parse(res);
          console.log("Mask applied. Status: ", envelope.result);
      }
  });
  ```

---

### `Window.CenterToScreen`
> Centers the given window to the screen.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.CenterToScreen",
          params: []
      })
  });
  ```

---

### `Window.CenterWithinParent`
> Centers the given window within its parent.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.CenterWithinParent",
          params: []
      })
  });
  ```

---

### `Window.Flash`
> Flashes the window and its taskbar icon.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.Flash",
          params: []
      })
  });
  ```

---

### `Window.GetValidWindowSizes`
> Returns the list of valid window sizes for the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.GetValidWindowSizes",
          params: []
      }),
      onSuccess: function(res) {
          const envelope = JSON.parse(res);
          const sizes = JSON.parse(envelope.result); // Double JSON parse required
          console.log("Available resolutions:", sizes);
      }
  });
  ```

---

### `Window.Hide`
> Hides the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.Hide",
          params: []
      })
  });
  ```

---

### `Window.Minimize`
> Minimizes the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.Minimize",
          params: []
      })
  });
  ```

---

### `Window.MoveBy`
> Moves the ux window by the given amount.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `deltaX` | Number | amount to adjust the ux window's horizontal position by |
  | `2` | `deltaY` | Number | amount to adjust the ux window's vertical position by |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.MoveBy",
          params: [150, -50] // Move 150px right, 50px up
      })
  });
  ```

---

### `Window.MoveTo`
> Moves the window to a specified horizontal/vertical position.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `x` | Number | X-coordinate to place the window on the screen |
  | `2` | `y` | Number | Y-coordinate to place the window on the screen |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.MoveTo",
          params: [200, 200]
      })
  });
  ```

---

### `Window.RemoveFromTaskbar`
> Removes the ux window from the taskbar.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.RemoveFromTaskbar",
          params: []
      })
  });
  ```

---

### `Window.ResizeBy`
> Resizes the ux window by the given width and height.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `deltaX` | Number | amount to adjust the ux window's width by |
  | `2` | `deltaY` | Number | amount to adjust the ux window's height by |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.ResizeBy",
          params: [100, 100] // Increase width and height by 100px
      })
  });
  ```

---

### `Window.ResizeTo`
> Resizes the ux window to the given width and height.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `width` | Number | width to resize the ux window to |
  | `2` | `height` | Number | height to resize the ux window to |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.ResizeTo",
          params: [1280, 720]
      })
  });
  ```

---

### `Window.Restore`
> Restores the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.Restore",
          params: []
      })
  });
  ```

---

### `Window.ScreenData`
> Returns the screen data for the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `queryId` | String | id of the persistent query |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.ScreenData",
          params: []
      }),
      onSuccess: function(res) {
          const envelope = JSON.parse(res);
          const data = JSON.parse(envelope.result); // Double JSON parse required
          console.log("Current window screen dimensions:", data);
      }
  });
  ```

---

### `Window.SetDebugWindowFrameVisibility`
> Sets the debug window's frame to the given visibility.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `visible` | Boolean | whether or not the native OS window frame should be visible |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.SetDebugWindowFrameVisibility",
          params: [true]
      })
  });
  ```

---

### `Window.SetResizeBounds`
> Sets the resize bounds of the the window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id of the window |
  | `1` | `minWidth` | Number | minimum width the window can be resized to, 0 = no min |
  | `2` | `minHeight` | Number | minimum height the window can be resized to, 0 = no min |
  | `3` | `maxWidth` | Number | maximum width the window can be resized to, 0 = no max |
  | `4` | `maxHeight` | Number | maximum height the window can be resized to, 0 = no max |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.SetResizeBounds",
          params: [1024, 576, 1920, 1080]
      })
  });
  ```

---

### `Window.SetTitle`
> Sets the ux window title to the given string.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `title` | String | the new title of the window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.SetTitle",
          params: ["League of Legends Client"]
      })
  });
  ```

---

### `Window.Show`
> Shows the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.Show",
          params: []
      })
  });
  ```

---

### `Window.ShowDevTools`
> Shows dev tools for the ux window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.ShowDevTools",
          params: []
      })
  });
  ```

---

### `Window.SyncMinimize`
> Sets the sync status if this window should minimize with the main window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | id of the ux window |
  | `1` | `enabled` | Boolean | if window should sync minimize with the main window |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Window.SyncMinimize",
          params: [true]
      })
  });
  ```

---

## 2. Mouse & Interaction (`Mouse.*`)
**Native Module:** `RiotInvoke_MouseModuleInit`

### `Mouse.SetDragBarHeight`
> Sets the client's drag bar height.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id of the window |
  | `1` | `height` | Number | height of the drag bar |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Mouse.SetDragBarHeight",
          params: [48]
      })
  });
  ```

---

### `Mouse.SetDragEnabled`
> Sets the ability to drag the window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id of the window |
  | `1` | `enabled` | Boolean | if dragging is enabled or disabled |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Mouse.SetDragEnabled",
          params: [true]
      })
  });
  ```

---

### `Mouse.SetResizeBounds`
> Sets the resize bounds of the the window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id of the window |
  | `1` | `minWidth` | Number | minimum width the window can be resized to |
  | `2` | `minHeight` | Number | minimum height the window can be resized to |
  | `3` | `maxWidth` | Number | maximum width the window can be resized to |
  | `4` | `maxHeight` | Number | maximum height the window can be resized to |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Mouse.SetResizeBounds",
          params: [1024, 576, 1920, 1080]
      })
  });
  ```

---

### `Mouse.SetResizeEnabled`
> Sets the ability to resize the window.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id of the window |
  | `1` | `enabled` | Boolean | if resizing is enabled or disabled |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "Mouse.SetResizeEnabled",
          params: [true]
      })
  });
  ```

---

## 3. File System Prompts (`File.*`)
**Native Module:** `RiotInvoke_FileModuleInit`

### `File.RequestDirectoryPath`
> Requests a directory path from the user, using the standard OS directory browser dialog.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id |
  | `1` | `initialPath` | String | path that dialog box opens to |
  | `2` | `windowText` | String | text displayed in dialog on windows and as window title on MacOS |
  | `3` | `okButtonText` | String | text to display in ok button |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "File.RequestDirectoryPath",
          params: [
              "C:\\Program Files", 
              "Select Folder Location", 
              "Select"
          ]
      }),
      onSuccess: function(res) {
          const envelope = JSON.parse(res);
          const dirPath = envelope.result; // Access directly as a raw string
          console.log("User selected directory:", dirPath);
      }
  });
  ```

---

### `File.RequestSaveFilePath`
> Requests a save file path from the user, using the standard OS save file dialog.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ux id |
  | `1` | `defaultName` | String | default file name |
  | `2` | `title` | String | title for the dialog |
  | `3` | `extensions` | Array | default file extensions (Pass as an array of wildcards, e.g. `["*.json"]`) |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "File.RequestSaveFilePath",
          params: [
              
              "plugin-config", 
              "Save Client Configuration", 
              [".json"] // Target is mapped to a std::vector in C++
          ]
      }),
      onSuccess: function(res) {
          const envelope = JSON.parse(res);
          const savePath = envelope.result; // Access directly as a raw string
          console.log("User selected save location:", savePath);
      }
  });
  ```

---

## 4. Riot Client & System (`RiotClient.*`)
**Native Module:** `RiotInvoke_RiotClientModuleInit`

### `RiotClient.Copy`
> Copies the currently selected text.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ID of the target UI context |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "RiotClient.Copy",
          params: []
      })
  });
  ```

---

### `RiotClient.Exit`
> Exits the application.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ID of the target UI context |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "RiotClient.Exit",
          params: []
      })
  });
  ```

---

### `RiotClient.Paste`
> Pastes the clipboard text.
* **Parameters:**
  | Index | Name | Type | Description |
  | :--- | :--- | :--- | :--- |
  | `0` | `uxId` | String | ID of the target UI context |

* **Example Call:**
  ```javascript
  window.riotInvoke({
      request: JSON.stringify({
          name: "RiotClient.Paste",
          params: []
      })
  });
  ```
---

## Disclaimer & License

This project is an independent reverse-engineering and documentation effort intended for educational, research, and plugin development purposes. All associated native layouts and designs are property of Riot Games.
---

*API Research and Documentation compiled by [@ReformedDoge](https://github.com/ReformedDoge).*



