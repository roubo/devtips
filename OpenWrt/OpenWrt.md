# OpenWrt NOTICES

## What's the "input device grab"

    By default, X input events are delivered to all clients that registered for a specific event on a window.
    The basic premise of a grab is that it affects event delivery to deliver events exclusively to one client only.
    There are three types of grabs, two classes of grabs, and two modes for a grab. The three types are:
    * active grabs
    * passive grabs
    * implicit passive grab
    The two classes are core grabs and device grabs and the two modes are synchronous and asynchronous.
    A grab comes in a combination of type + class + mode, so a grab may be an "active synchronous device grab".

## Something about the "Blcontrol problem"

    Since the input device (here is a touch screen) can not be grab by two clients with the block mode.
    And it is hard to catch the events by realtime in user space, so I processes the events in kernel space.
    Some ioctl are usefull.
    see -> [modou fb driver]

## How to make code simple

    When I feel that it is hard to manage a project, and the project may be not good in the beginning.
    Simple things are not necessarily the best, but the best one must be simple.
    Every function must be used by many others.
    Every file must be used by many others.
    If not, it is failure.
    See -> [app-ss-vpn ss.sh](https://gitcafe.com/Modou/app-ss-vpn/blob/master/sbin/ss.sh)

## About Lua coroutine

    luci cgi run function:
    ```lua
    function run()
      local r = luci.http.Request(
        luci.sys.getenv(),
        limitsource(io.stdin, tonumber(luci.sys.getenv("CONTENT_LENGTH"))),
        ltn12.sink.file(io.stderr)
      )
      local x = coroutine.create(luci.dispatcher.httpdispatch)
      local hcache = ""
      local active = true
      while coroutine.status(x) ~= "dead" do
        local res, id, data1, data2 = coroutine.resume(x, r)
        if not res then
          print("Status: 500 Internal Server Error")
          print("Content-Type: text/plain\n")
          print(id)
          break;
        end
        if active then
          if id == 1 then
            io.write("Status: " .. tostring(data1) .. " " .. data2 .. "\r\n")
          elseif id == 2 then
            hcache = hcache .. data1 .. ": " .. data2 .. "\r\n"
          elseif id == 3 then
            io.write(hcache)
            io.write("\r\n")
          elseif id == 4 then
            io.write(tostring(data1 or ""))
          elseif id == 5 then
            io.flush()
            io.close()
            active = false
          elseif id == 6 then
            data1:copyz(nixio.stdout, data2)
            data1:close()
          end
        end
      end
    end
    ```
