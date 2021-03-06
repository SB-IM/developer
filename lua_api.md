# Lua api

> Lua version: 5.1

## Main

For Example:

```lua
function main(plan)
end
```

The function `main()` params is `plan`

## Plan

This is Plan, Only `main()` params

```lua
plan:CleanDialog()

ask_status = {
  name = "ARE YOU OK ?",
  buttons = {
    {name = "Fine, thank you.", message = 'fine', level = 'primary'},
    {name = "I feel bad.", message = 'bad', level = 'danger'},
  }
}
plan:ToggleDialog(ask_status)

msg, err = plan:Gets()
if err ~= nil then
  print(msg)
  print(json.encode(err))
end

plan:CleanDialog()

print(plan:FileUrl("file"))
```

### ToggleDialog(table)

Topic: `plans/:id/dialog`

Send a confirmation form

[params table reference](https://developer.sb.im/#/mqtt?id=dialog)

```lua
ask_status = {
  name = "ARE YOU OK ?",
  buttons = {
    {name = "Fine, thank you.", message = 'fine', level = 'primary'},
    {name = "I feel bad.", message = 'bad', level = 'danger'},
  }
}
plan:ToggleDialog(ask_status)
```

### CleanDialog()

Close Dialog

Topic: `plans/:id/dialog` message `{}`

```lua
plan:CleanDialog()
```

### Gets() string

get `plans/:id/term` message

```lua
print(plan:Gets())

msg, err = plan:Gets()
if err ~= nil then
  print(msg)
  print(json.encode(err))
end
```

### Puts(string)

put `plans/:id/term` message

```lua
plan:Puts("test")
sleep("1s")
plan:Puts("test2")
```

### Notification(msg string, level = 5)

```lua
plan:Notification("notification")

plan:Notification("notification", 3)
```

### nodeID

> The id of this current plan need `node`

### GetExtra() extra map[string]string

Get Plan Extra `key/value`

This `extra` is `map[string]string` of `table`

### SetExtra(extra map[string]string)

Set Plan Extra `key/value`

```lua
print("Extra:", json.encode(plan:GetExtra()))

local extra = plan:GetExtra()
extra["xxx"] = "aa"
extra["ccc"] = "aaa"
plan:SetExtra(extra)
```

### GetJobExtra() extra map[string]string

Get Plan Job Extra `key/value`

### SetJobExtra(extra map[string]string)

Set Plan Job Extra `key/value`

```lua
local job_extra = plan:GetJobExtra()
job_extra["ttt"] = "xxx"
print("Job Extra:", json.encode(job_extra))
plan:SetJobExtra(job_extra)
```

### GetFileContent(key string) (filename, content string)

Get Plan Files Content

### SetFileContent(key, filename, content string)

Set Plan Files Content

```lua
print("Files:", json.encode(plan:GetFiles()))

xpcall(function()
  print(plan:GetFileContent("test_files"))
end,
function()
  plan:SetFileContent("test_files", "test.txt", "233")
end)

local filename, content = plan:GetFileContent("test_files")
if content == "233" then
  plan:SetFileContent("test_files", "test2.txt", "456")
else
  plan:SetFileContent("test_files", "test.txt", "233")
end
print(plan:GetFileContent("test_files"))
```

### GetJobFileContent(key string) (filename, content string)

Get Plan Job Files Content

### SetJonFileContent(key, filename, content string)

Set Plan Job Files Content

```lua
print("Job Files:", json.encode(plan:GetJobFiles()))

xpcall(function()
  print(plan:GetJobFileContent("test_files"))
end,
function()
  plan:SetJobFileContent("test_files", "test.txt", "233")
end)

local filename, content = plan:GetJobFileContent("test_files")
if content == "233" then
  plan:SetJobFileContent("test_files", "test2.txt", "456")
else
  plan:SetJobFileContent("test_files", "test.txt", "233")
end
print(plan:GetJobFileContent("test_files"))
```

### FileUrl(key string)

```lua
print("Blobs:", plan:FileUrl("test_files"))
print("Blobs:", plan:FileUrl("test_blobs"))
```

### JobFileUrl(key string)

```lua
print("Job Blobs:", plan:JobFileUrl("test_files"))
print("Job Blobs:", plan:JobFileUrl("test_blobs"))
```

## Class node

```lua
local drone_id = plan.nodeID
local drone = NewNode(drone_id)

local depot_id = drone:GetID()
local depot = NewNode(depot_id)

xpcall(function()
  local promise = depot:AsyncCall("wait_to_boot_finish")
  depot:SyncCall("power_on_drone")
  depot:SyncCall("power_on_remote")

  -- Block: get rpc result
  local result = promise()

end, function()
  drone:SyncCall("emergency_stop")
end)

local data = drone:GetMsg("battery")

```

### SyncCall(string [, table]) table

> Sync jsonrpc call

### AsyncCall(string [, table]) function() table

> Async jsonrpc call

return a function

### GetID([string]) string

default params: `link_id`

`GetID() == GetID("link_id")`

### GetStatus() table

`nodes/:id/status`

### GetMsg(string) table

`nodes/:id/msg/+`

## Log

### NewLog([fnLine, fnWord function])

```lua
local log = NewLog()

-- Or

local log = NewLog(function(line, nu)
  return tostring(nu) .. ": \t" .. os.date("%Y/%m/%d %H:%M:%S") .. " " .. line
end)

-- Or

local log = NewLog(function(line, nu)
  return tostring(nu) .. ": \t" .. os.date("%Y/%m/%d %H:%M:%S") .. " " .. line
end,
function(word, nu)
  if type(word) == "table" then
    return json.encode(word)
  end
  return tostring(word)
end)
```

### fnLine(string, number) string

> Every line hook function

Default: if `fnLine == nil`

```lua
if fnLine == nil then
  fnLine = function(line, nu)
    return tostring(nu) .. ": " .. line
  end
end
```

### fnWord(any, number) string

> Every word hook function

Default: if `fnWord == nil`

```lua
if fnWord == nil then
  fnWord = function(word, nu)
    return tostring(word)
  end
end
```

### Println(...)

> Print auto add `\n`

```lua
log:Println("xxxxx")
log:Println("xxxxx", "xxxxx", "xxxxx", "xxxxx")
log:Println(1, "xxxxx")
print(log:GetContent())
```

```lua
function test_log()
  local log = NewLog()
  log:Println("xxxxx")
  log:Println("xxxxx")
  log:Println("xxxxx", "xxxxx")
  log:Println("xxxxx", "xxxxx", "xxxxx")
  log:Println("xxxxx", "xxxxx", "xxxxx", "xxxxx")
  log:Println(1, "xxxxx")
  print(log:GetContent())
end

function test_logfn()
  local log = NewLog(function(line, nu)
    return tostring(nu) .. ": \t" .. os.date("%Y/%m/%d %H:%M:%S") .. " " .. line
  end)
  log:Println("xxxxx")
  log:Println(1, "xxxxx")
  for i=10,1,-1 do
    log:Println(i, "ccccc")
  end
  print(log:GetContent())
end
```

### GetContent() string

> Get All Content

```lua
_raw_print = print
local log = NewLog(function(line, nu)
  return tostring(nu) .. ": \t" .. os.date("%Y/%m/%d %H:%M:%S") .. " " .. line
end,
function(word, nu)
  if type(word) == "table" then
    return json.encode(word)
  end
  return tostring(word)
end)
print = function(...)
  _raw_print(unpack(arg))
  log:Println(unpack(arg))
end

function main(plan)
  print("=== START Lua ===")

  -- Your workflow

  print("=== END Lua END ===")
  plan:SetJobFileContent("luavm", "luavm.txt", log:GetContent())
end
```

## Geo

Geography util. a Object

### Distance(aLng, aLat, bLng, bLat number) number

2D Distance measurement

`Geo:Distance(aLng, aLat, bLng, bLat)`

```lua
-- 114.2247765, 22.6857991
-- 114.22475167, 22.68580217
-- = 2.57202994
local distance = Geo:Distance(114.2247765, 22.6857991, 114.22475167, 22.68580217)
if math.floor(distance) == 2 then
  print("Distance:", distance)
else
  error("Distance sum error:", distance)
end
```

## sleep(string)

```lua
sleep("1ms")
```

such as "300ms", "-1.5h" or "2h45m".
Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h" [golang time#ParseDuration](https://golang.org/pkg/time/#ParseDuration)

