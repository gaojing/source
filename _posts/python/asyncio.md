title: asyncio
---

# Base Event Loop

## Run an event loop
	AbstractEventLoop.run_forever() #直到调用stop()后停止
	AbstractEventLoop.run_until_complete(future) #直到future完成为止

## Calls 
	AbstractEventLoop.call_soon(callback, *args) #尽快调用callback

## Delayed calls
	AbstractEventLoop.call_later(delay, callback, *args) #延时后调用callback