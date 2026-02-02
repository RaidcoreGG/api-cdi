\page resource-sharing Shared Resources Tutorial
\warning
This page is currently incomplete, further updates will come at a later date.

Nexus provides an API for sharing resources with other addons.

## Sharing a resource
As an example, this guide will detail how to share a pointer to a struct containing both data and functions. 

### `types.hpp`
```cpp
namespace ExampleAddon {
	struct Data {
		int Version;
		void (*LogGreeting)();
	}
}
```
### `DataLink.hpp`
```cpp
#include "types.hpp"
namespace Data {
	void setupDatalink();
	void LogGreeting();
	void cleanupDatalink();
}
```
### `DataLink.cpp`
```cpp
#include "DataLink.hpp"
ExampleAddon::Data* _DATA;
void Data::LogGreeting() {
	Addon_API->Log(LOGL_INFO, "exampleaddon", "Hello, World!");
}
void Data::setupDatalink() {
	_DATA = Addon_API->ShareResource("DL_EXAMPLEADDON_DATA", sizeof(ExampleAddon::Data));
	_DATA->Version = 3
	_DATA->LogGreeting = Data::LogGreeting;
}
void Data::cleanupDatalink() {
	_DATA->LogGreeting = nullptr;
}
```