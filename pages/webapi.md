\page webapi api.raidcore.gg
api.raidcore.gg supports the following endpoints

### `GET api.raidcore.gg/addonlibrary`
returns an array of JSON objects matching the following type
```ts
interface Addon {
	id: number;
	name: string;
	author: string;
	description: string;
	download: URL;
	filename: string;
	addon_policy_tier: number;
	tags: string[];
}
```

### `GET api.raidcore.gg/arcdpslibrary`
returns an array of objects matching the following type
```ts
interface Addon {
	id: number;
	name: string;
	author: string;
	description: string;
	download: URL;
	filename: string;
	addon_policy_tier: number;
	tags: string[];
}
```

### `GET api.raidcore.gg/nexusversion`
returns an object matching the following type
```ts
interface NexusVersion{
	Major: number;
	Minor: number;
	Build: number;
	Revision: number;
	Changelog: string;
}
```