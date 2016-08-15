# Gateway_LL Requirements

## Overview
The Gateway_LL API to used to create and manage a gateway. Modules are added to the gateway, and communication "links" are established between modules via a message broker.

## References

## Gateway Handle Implementation

This section details the internal structure defined in the gateway implementation used to track a gateway and the information it contains.

```
typedef struct GATEWAY_HANDLE_DATA_TAG {
	/** @brief Vector of MODULE_DATA modules that the Gateway must track */
	VECTOR_HANDLE modules;

    /** @brief The message broker used by this gateway */
    BROKER_HANDLE broker;    

	/** @brief Vector of LINK_DATA links that the Gateway must track */
	VECTOR_HANDLE links;
} GATEWAY_HANDLE_DATA;
```

## Exposed API
```
#ifndef GATEWAY_LL_H
#define GATEWAY_LL_H

#include "azure_c_shared_utility/macro_utils.h"
#include "azure_c_shared_utility/vector.h"
#include "module.h"

#ifdef __cplusplus
extern "C"
{
#endif



#define GATEWAY_ADD_LINK_RESULT_VALUES \
    GATEWAY_ADD_LINK_SUCCESS, \
    GATEWAY_ADD_LINK_ERROR, \
    GATEWAY_ADD_LINK_INVALID_ARG

/** @brief	Enumeration describing the result of ::Gateway_LL_AddLink.
*/
DEFINE_ENUM(GATEWAY_ADD_LINK_RESULT, GATEWAY_ADD_LINK_RESULT_VALUES);

/** @brief Struct representing a single link for a gateway. */
typedef struct GATEWAY_LINK_ENTRY_TAG
{
	/** @brief The name of the module which is going to send messages. */
	const char* module_source;

	/** @brief The name of the module which is going to receive messages. */
	const char* module_sink;
} GATEWAY_LINK_ENTRY;

/** @brief Struct representing a particular gateway. */
typedef struct GATEWAY_HANDLE_DATA_TAG* GATEWAY_HANDLE;

/** @brief Struct representing a single entry of the #GATEWAY_PROPERTIES. */
typedef struct GATEWAY_MODULES_ENTRY_TAG
{
	/** @brief The (possibly @c NULL) name of the module */
	const char* module_name;
	
	/** @brief The path to the .dll or .so of the module */
	const char* module_path;
	
	/** @brief The user-defined configuration object for the module */
	const void* module_configuration;
} GATEWAY_MODULES_ENTRY;

/** @brief	Struct representing the properties that should be used when 
			creating a module; each entry of the @c VECTOR_HANDLE being a 
*			#GATEWAY_MODULES_ENTRY. 
*/
typedef struct GATEWAY_PROPERTIES_DATA_TAG
{
	/** @brief Vector of #GATEWAY_MODULES_ENTRY objects. */
	VECTOR_HANDLE gateway_modules;

	/** @brief Vector of #GATEWAY_LINK_ENTRY objects. */
	VECTOR_HANDLE gateway_links;
} GATEWAY_PROPERTIES;



/** @brief		Creates a new gateway using the provided #GATEWAY_PROPERTIES.
*
*	@param		properties		#GATEWAY_PROPERTIES structure containing 
*								specific module properties and information.
*
*	@return		A non-NULL #GATEWAY_HANDLE that can be used to manage the 
*				gateway or @c NULL on failure.
*/
extern GATEWAY_HANDLE Gateway_LL_Create(const GATEWAY_PROPERTIES* properties);

/** @brief		Destroys the gateway and disposes of all associated data.
*
*	@param		gw		#GATEWAY_HANDLE to be destroyed.
*/
extern void Gateway_LL_Destroy(GATEWAY_HANDLE gw);

/** @brief		Creates a new module based on the GATEWAY_MODULES_ENTRY*.
*
*	@param		gw		Pointer to a #GATEWAY_HANDLE to add the Module onto.
*	@param		entry	Pointer to a #GATEWAY_MODULES_ENTRY structure
*						describing the module.
*
*	@return		A non-NULL #MODULE_HANDLE to the newly created and added
*				Module, or @c NULL on failure.
*/
extern MODULE_HANDLE Gateway_LL_AddModule(GATEWAY_HANDLE gw, const GATEWAY_MODULES_ENTRY* entry);



/** @brief		Removes the provided module from the gateway and all links that involves this module.
*
*	@param		gw		Pointer to a #GATEWAY_HANDLE from which to remove the
*						Module.
*	@param		module	Pointer to a #MODULE_HANDLE that needs to be removed.
*/
extern void Gateway_LL_RemoveModule(GATEWAY_HANDLE gw, MODULE_HANDLE module);

/** @brief		Adds a link to a gateway message broker.
*
*	@param		gw		    Pointer to a #GATEWAY_HANDLE from which link is going to be added.
*
*	@param		entryLink	Pointer to a #GATEWAY_LINK_ENTRY to be added.
*
*	@return		A GATEWAY_ADD_LINK_RESULT with the operation result.
*/
extern GATEWAY_ADD_LINK_RESULT Gateway_LL_AddLink(GATEWAY_HANDLE gw, const GATEWAY_LINK_ENTRY* entryLink);

/** @brief		Remove a link from a gateway message broker.
*
*	@param		gw		    Pointer to a #GATEWAY_HANDLE from which link is going to be removed.
*
*	@param		entryLink	Pointer to a #GATEWAY_LINK_ENTRY to be removed.
*/
extern void Gateway_LL_RemoveLink(GATEWAY_HANDLE gw, const GATEWAY_LINK_ENTRY* entryLink);

#ifdef UWP_BINDING

/** @brief		Creates a new gateway using the provided #MODULEs and #BROKER_HANDLE.
*
*	@param		modules   		#VECTOR_HANDLE structure containing
*								specific modules.
*
*	@param		broker     		#BROKER_HANDLE structure containing
*								specific message broker.
*
*	@return		A non-NULL #GATEWAY_HANDLE that can be used to manage the
*				gateway or @c NULL on failure.
*/
extern GATEWAY_HANDLE Gateway_LL_UwpCreate(const VECTOR_HANDLE modules, broker);

/** @brief		Destroys the gateway and disposes of all associated data.
*
*	@param		gw		#GATEWAY_HANDLE to be destroyed.
*/
extern void Gateway_LL_UwpDestroy(GATEWAY_HANDLE gw);

#endif // UWP_BINDING

#ifdef __cplusplus
}
#endif

#endif // GATEWAY_LL_H
```

## Gateway_Create
```
extern GATEWAY_HANDLE Gateway_LL_Create(const GATEWAY_PROPERTIES* properties);
```
Gateway_LL_Create creates a new gateway using information from the `GATEWAY_PROPERTIES` struct to create modules and associate them with a message broker.

**SRS_GATEWAY_LL_14_001: [** This function shall create a `GATEWAY_HANDLE` representing the newly created gateway. **]**

**SRS_GATEWAY_LL_14_002: [** This function shall return `NULL` upon any memory allocation failure. **]**

**SRS_GATEWAY_LL_14_003: [** This function shall create a new `BROKER_HANDLE` for the gateway representing this gateway's message broker. **]**

**SRS_GATEWAY_LL_14_004: [** This function shall return `NULL` if a `BROKER_HANDLE` cannot be created. **]**

**SRS_GATEWAY_LL_14_033: [** The function shall create a vector to store each `MODULE_DATA`. **]**

**SRS_GATEWAY_LL_04_001: [** The function shall create a vector to store each `LINK_DATA` **]**

**SRS_GATEWAY_LL_14_034: [** This function shall return `NULL` if a `VECTOR_HANDLE` cannot be created. **]**

**SRS_GATEWAY_LL_14_035: [** This function shall destroy the previously created `BROKER_HANDLE` and free the `GATEWAY_HANDLE` if the `VECTOR_HANDLE` cannot be created. **]**

**SRS_GATEWAY_LL_14_009: [** The function shall use each of `GATEWAY_PROPERTIES`'s `gateway_modules` to create and add a module to the gateway's message broker. **]**

**SRS_GATEWAY_LL_14_036: [** If any `MODULE_HANDLE` is unable to be created from a `GATEWAY_MODULES_ENTRY` the `GATEWAY_HANDLE` will be destroyed. **]**

**SRS_GATEWAY_LL_04_004: [** If a module with the same `module_name` already exists, this function shall fail and the `GATEWAY_HANDLE` will be destroyed. **]**

**SRS_GATEWAY_LL_04_002: [** The function shall use each `GATEWAY_LINK_ENTRY` of `GATEWAY_PROPERTIES`'s `gateway_links` to add a `LINK` to `GATEWAY_HANDLE`'s broker. **]**

**SRS_GATEWAY_LL_26_001: [** This function shall initialize attached Event System and report `GATEWAY_CREATED` event. **]**

**SRS_GATEWAY_LL_26_002: [** If Event System module fails to be initialized the gateway module shall be destroyed and NULL returned with no events reported. **]**

**SRS_GATEWAY_LL_26_010: [** This function shall report `GATEWAY_MODULE_LIST_CHANGED` event. **]**

**SRS_GATEWAY_LL_04_003: [** If any `GATEWAY_LINK_ENTRY` is unable to be added to the broker the `GATEWAY_HANDLE` will be destroyed. **]**

```
extern GATEWAY_HANDLE Gateway_LL_UwpCreate(const VECTOR_HANDLE modules, BROKER_HANDLE broker);
```
Gateway_LL_UwpCreate creates a new gateway using modules in the `VECTOR_HANDLE` struct and the message broker described by the `BROKER_HANDLE` to configure the modules on the message broker.

**SRS_GATEWAY_LL_99_001: [** This function shall create a `GATEWAY_HANDLE` representing the newly created gateway. **]**

**SRS_GATEWAY_LL_99_002: [** This function shall return `NULL` upon any memory allocation failure. **]**

**SRS_GATEWAY_LL_99_003: [** This function shall return `NULL` if a `NULL` `BROKER_HANDLE` is received. **]**

**SRS_GATEWAY_LL_99_004: [** This function shall return `NULL` if a `NULL` `VECTOR_HANDLE` is received. **]**

**SRS_GATEWAY_LL_99_005: [** The function shall increment the BROKER_HANDLE reference count if the MODULE_HANDLE was successfully linked to the GATEWAY_HANDLE_DATA's message broker. **]**

## Gateway_Destroy
```
extern void Gateway_LL_Destroy(GATEWAY_HANDLE gw);
```
Gateway_LL_Destroy destroys a gateway represented by the `gw` parameter.

**SRS_GATEWAY_LL_14_005: [** If `gw` is `NULL` the function shall do nothing. **]**

**SRS_GATEWAY_LL_14_028: [** The function shall remove each module in `GATEWAY_HANDLE_DATA`'s `modules` vector and destroy `GATEWAY_HANDLE_DATA`'s `modules`. **]**

**SRS_GATEWAY_LL_04_014: [** The function shall remove each link in `GATEWAY_HANDLE_DATA`'s `links` vector and destroy `GATEWAY_HANDLE_DATA`'s `link`. **]**

**SRS_GATEWAY_LL_14_037: [** If `GATEWAY_HANDLE_DATA`'s message broker cannot remove a module, the function shall log the error and continue removing modules from the `GATEWAY_HANDLE`. **]**

**SRS_GATEWAY_LL_14_006: [** The function shall destroy the `GATEWAY_HANDLE_DATA`'s `broker` `BROKER_HANDLE`. **]**

**SRS_GATEWAY_LL_26_003: [** If the Event System module is initialized, this function shall report `GATEWAY_DESTROYED` event. **]**

**SRS_GATEWAY_LL_26_004: [** This function shall destroy the attached Event System.  **]**

```
extern void Gateway_LL_UwpDestroy(GATEWAY_HANDLE gw);
```
Gateway_LL_UwpDestroy destroys a gateway represented by the `gw` parameter.

**SRS_GATEWAY_LL_99_006: [** If `gw` is `NULL` the function shall do nothing. **]**

**SRS_GATEWAY_LL_99_007: [** The function shall detach modules from the `GATEWAY_HANDLE_DATA`'s `broker` `BROKER_HANDLE`. **]**

**SRS_GATEWAY_LL_99_008: [** If `GATEWAY_HANDLE_DATA`'s `broker` cannot detach a module, the function shall log the error and continue unloading the module from the `GATEWAY_HANDLE`. **]**

**SRS_GATEWAY_LL_99_009: [** The function shall decrement the BROKER_HANDLE reference count. **]**

**SRS_GATEWAY_LL_99_010: [** The function shall destroy the `GATEWAY_HANDLE_DATA`'s `broker` `BROKER_HANDLE`. **]**

## Gateway_AddModule
```
extern MODULE_HANDLE Gateway_LL_AddModule(GATEWAY_HANDLE gw, const GATEWAY_PROPERTIES_ENTRY* entry);
```
Gateway_LL_AddModule adds a module to the gateway's message broker using the provided `GATEWAY_PROPERTIES_ENTRY`'s `module_path` and `GATEWAY_PROPERTIES_ENTRY`'s `module_properties`.

**SRS_GATEWAY_LL_14_011: [** If `gw`, `entry`, or `GATEWAY_MODULES_ENTRY`'s `module_path` is `NULL` the function shall return `NULL`. **]**

**SRS_GATEWAY_LL_14_012: [** The function shall load the module located at `GATEWAY_MODULES_ENTRY`'s `module_path` into a `MODULE_LIBRARY_HANDLE`. **]**

**SRS_GATEWAY_LL_14_031: [** If unsuccessful, the function shall return `NULL`. **]**

**SRS_GATEWAY_LL_14_013: [** The function shall get the `const MODULE_APIS*` from the `MODULE_LIBRARY_HANDLE`. **]**

**SRS_GATEWAY_LL_14_015: [** The function shall use the `MODULE_APIS` to create a `MODULE_HANDLE` using the `GATEWAY_MODULES_ENTRY`'s `module_properties`. **]**

**SRS_GATEWAY_LL_14_016: [** If the module creation is unsuccessful, the function shall return `NULL`. **]**

**SRS_GATEWAY_LL_14_017: [** The function shall attach the module to the `GATEWAY_HANDLE_DATA`'s `broker` using a call to `Broker_AddModule`. **]**

**SRS_GATEWAY_LL_14_039: [** The function shall increment the `BROKER_HANDLE` reference count if the `MODULE_HANDLE` was successfully linked to the `GATEWAY_HANDLE_DATA`'s `broker`. **]**

**SRS_GATEWAY_LL_14_018: [** If the function cannot attach the module to the message broker, the function shall return `NULL`. **]**

**SRS_GATEWAY_LL_14_029: [** The function shall create a new `MODULE_DATA` containting the `MODULE_HANDLE` and `MODULE_LIBRARY_HANDLE` if the module was successfully linked to the message broker. **]**

**SRS_GATEWAY_LL_14_032: [** The function shall add the new `MODULE_DATA` to `GATEWAY_HANDLE_DATA`'s `modules` if the module was successfully linked to the message broker. **]**

**SRS_GATEWAY_LL_14_030: [** If any internal API call is unsuccessful after a module is created, the library will be unloaded and the module destroyed. **]**

**SRS_GATEWAY_LL_14_019: [** The function shall return the newly created `MODULE_HANDLE` only if each API call returns successfully. **]**

**SRS_GATEWAY_LL_99_011: [** The function shall assign `module_apis` to `MODULE::module_apis`. **]**

## Gateway_RemoveModule
```
extern void Gateway_LL_RemoveModule(GATEWAY_HANDLE gw, MODULE_HANDLE module);
```
Gateway_RemoveModule will remove the specified `module` from the message broker.

**SRS_GATEWAY_LL_14_020: [** If `gw` or `module` is `NULL` the function shall return. **]**

**SRS_GATEWAY_LL_14_023: [** The function shall locate the `MODULE_DATA` object in `GATEWAY_HANDLE_DATA`'s `modules` containing `module` and return if it cannot be found.  **]**

**SRS_GATEWAY_LL_14_021: [** The function shall detach `module` from the `GATEWAY_HANDLE_DATA`'s `broker` `BROKER_HANDLE`. **]**

**SRS_GATEWAY_LL_14_022: [** If `GATEWAY_HANDLE_DATA`'s `broker` cannot detach `module`, the function shall log the error and continue unloading the module from the `GATEWAY_HANDLE`. **]**

**SRS_GATEWAY_LL_14_038: [** The function shall decrement the `BROKER_HANDLE` reference count. **]**

**SRS_GATEWAY_LL_14_024: [** The function shall use the `MODULE_DATA`'s `library_handle` to retrieve the `MODULE_APIS` and destroy `module`. **]**

**SRS_GATEWAY_LL_14_025: [** The function shall unload `MODULE_DATA`'s `library_handle`. **]**

**SRS_GATEWAY_LL_14_026: [** The function shall remove that `MODULE_DATA` from `GATEWAY_HANDLE_DATA`'s `modules`. **]**

## Gateway_AddEventCallback
```
extern void Gateway_AddEventCallback(GATEWAY_HANDLE gw, GATEWAY_EVENT event_type, GATEWAY_CALLBACK callback);
```
Gateway_AddEventCallback registers callback function to listen to some kind of `GATEWAY_EVENT`.
When the event happens the callback will be put in a queue and executed in a seperate callback thread in First-In-First-Out order of registration for that event
Also see `event_system_requirements.md` file for further requirements.

**SRS_GATEWAY_LL_26_006: [** This function shall log a failure and do nothing else when `gw` parameter is NULL. **]**

## Gateway_GetModuleList
```
extern VECTOR_HANDLE Gateway_GetModuleList(GATEWAY_HANDLE gw);
```

**SRS_GATEWAY_LL_26_007: [** This function shall return a snapshot copy of information about current gateway modules. **]**

**SRS_GATEWAY_LL_26_008: [** If the `gw` parameter is NULL, the function shall return NULL handle and not allocate any data. **]**

**SRS_GATEWAY_LL_26_009: [** This function shall return a NULL handle should any internal callbacks fail. **]**


## Gateway_LL_AddLink
```
extern GATEWAY_ADD_LINK_RESULT Gateway_LL_AddLink(GATEWAY_HANDLE gw, const GATEWAY_LINK_ENTRY* entryLink);
```
Gateway_LL_AddLink adds a link to the gateway's message broker using the provided `GATEWAY_LINK_ENTRY`'s `module_path` and `GATEWAY_PROPERTIES_ENTRY`'s `module_properties`.

**SRS_GATEWAY_LL_04_008: [** If `gw` , `entryLink`, `entryLink->module_source` or `entryLink->module_source` is NULL the function shall return `GATEWAY_ADD_LINK_INVALID_ARG`. **]**

**SRS_GATEWAY_LL_04_009: [** This function shall check if a given link already exists.  **]**

**SRS_GATEWAY_LL_04_010: [** If the entryLink already exists it the function shall return `GATEWAY_ADD_LINK_ERROR` **]**

**SRS_GATEWAY_LL_04_011: [** If the module referenced by the `entryLink->module_source` or `entryLink->module_sink` doesn't exists this function shall return `GATEWAY_ADD_LINK_ERROR` **]**

**SRS_GATEWAY_LL_04_012: [** This function shall add the entryLink to the `gw->links` **]**

**SRS_GATEWAY_LL_04_013: [** If adding the link succeed this function shall return `GATEWAY_ADD_LINK_SUCCESS` **]**

## Gateway_LL_RemoveLink
```
extern void Gateway_LL_RemoveLink(GATEWAY_HANDLE gw, const GATEWAY_LINK_ENTRY* entryLink);
```
Gateway_RemoveLink will remove the specified `link` from the message broker.

**SRS_GATEWAY_LL_04_005: [** If `gw` or `entryLink` is `NULL` the function shall return. **]**  

**SRS_GATEWAY_LL_04_006: [** The function shall locate the `LINK_DATA` object in `GATEWAY_HANDLE_DATA`'s `links` containing `link` and return if it cannot be found. **]**

**SRS_GATEWAY_LL_04_007: [** The functional shall remove that `LINK_DATA` from `GATEWAY_HANDLE_DATA`'s `links`. **]**