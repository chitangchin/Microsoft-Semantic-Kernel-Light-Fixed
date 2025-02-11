# Fixing Microsofts Semantic Kernel Lights Demo

## 1. Adding Required attribute to LightModel Class Property Name

### Issue: 

Non-nullable property 'Name' must contain a non-null value when exiting constructor. Consider adding the 'required' modifier or declaring the property as nullable.CS8618

### Proposal

LightsPlugin.cs - Line: 46

Original:
   ```
   [JsonPropertyName("name")]
   public string Name { get; set; }
   ```

Updated:
   ```
   [JsonPropertyName("name")]
   public required string Name { get; set; }
   ```
### Why:

String property name must not allow null

## 2. Providing Null forgiving operator to UserInput

### Issue:

Possible null reference argument for parameter 'content' in 'void ChatHistory.AddUserMessage(string content)'.CS8604

### Proposal

Program.cs - Line: 43

Original:
```
    history.AddUserMessage(userInput);
```

Updated:
```
    history.AddUserMessage(userInput!);
```

### Why:

While loop condition prevents userInput from being null, therefore we can provide a null forgiving operator at line 43.

## 3. Unecessary Async functions

### Issue:

This async method lacks 'await' operators and will run synchronously. Consider using the 'await' operator to await non-blocking API calls, or 'await Task.Run(...)' to do CPU-bound work on a background thread.CS1998

### Proposal

Program.cs - Line: 17 - 38

Original:
```
   [KernelFunction("get_lights")]
   [Description("Gets a list of lights and their current state")]
   public async Task<List<LightModel>> GetLightsAsync()
   {
      return lights;
   }

   [KernelFunction("change_state")]
   [Description("Changes the state of the light")]
   public async Task<LightModel?> ChangeStateAsync(int id, bool isOn)
   {
      var light = lights.FirstOrDefault(light => light.Id == id);

      if (light == null)
      {
         return null;
      }

      // Update the light with the new state
      light.IsOn = isOn;

      return light;
   }

```

Updated:
```
    [KernelFunction("get_lights")]
    [Description("Gets a list of lights and their current state")]
    public async Task<List<LightModel>> GetLightsAsync()
    {
        await Task.CompletedTask;
        return lights;
    }

    [KernelFunction("change_state")]
    [Description("Changes the state of the light")]
    public async Task<LightModel?> ChangeStateAsync(int id, bool isOn)
    {
        await Task.CompletedTask; // Dummy await

        var light = lights.FirstOrDefault(light => light.Id == id);

        if (light == null)
        {
            return null;
        }

        light.IsOn = isOn;
        return light;
    }
```

### Why:

We should await for task completion within our async function
