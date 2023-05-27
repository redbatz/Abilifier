# Abilifier

**Versions 1.2.0.4 and higher require modtek v3 or higher**

**Versions 1.4.0.0 amd higher require IRBTModUtils, available here: https://github.com/BattletechModders/IRBTModUtils/releases**

This is a mod for HBS Battletech that allows developers to give players <b>choices</b> when it comes to leveling up abilities. Special thanks to kMiSSioN for allowing the reuse of their code adding additional ability buttons to the Combat HUD.

To use, place your new AbilityDefs in an appropriate subfolder, either in the Abilifier folder or elsewhere in your modpack, and ensure you have the appropriate manifest entry in your mod.json so modtek will load the new AbilityDefs, e.g.
```
 "Manifest": [
		{ "Type": "AbilityDef", "Path": "Abilities" },
```
<b>In addition</b>, you must add the new Ability Ids to `SimGameConstants` under the `Progression` entry, e.g.:
```
	"Progression" : {
		"GunnerySkills" :[
			[
				"TraitDefWeaponHit1"
			],
			[
				"TraitDefWeaponHit2"
			],
			[
				"TraitDefWeaponHit3"
			],
			[
				"TraitDefWeaponHit4"
			],
			[
				"TraitDefWeaponHit5",
				"AbilityDefG5",
				"AbilityDefG5a"
			],
```
Using the above, a player reaching Gunnery 5 will have the choice between the abilities with Ids `AbilityDefG5` and `AbilityDefG5a`. Using the existing naming scheme (appending "a", "b", etc. for new abilities in a skill tree) is not required, but is recommended for potential future compatibility.

Lastly, developers have a choice between the display style for the ability selection pop-up. The default method is to display the entire body of the Ability details in the AbilityDef .json, e.g:
![TextPop](https://github.com/BattletechModders/Abilifier/blob/master/doc/textpopup.png)

The alternative style is to use a hover tooltip, but this requires a little extra work on the developers' part:
![HoverPop](https://github.com/BattletechModders/Abilifier/blob/master/doc/tooltippopup.png)

To use a hover tooltip, you will need to create BaseDescriptionDef (essentially lore hover items) .jsons for <b>all</b> abilities, including any vanilla abilities being used, and place them in `Abilifier/AbilityDescs`. The BaseDescriptionDef Id <b>must</b> be identical to the AbilityDef Id, ending in "Desc" as below: 
```
{
    "Id" : "AbilityDefP5aDesc",
    "Name" : "MOSTEST SUREST FOOTING",
    "Details" : "PASSIVE: 'Mechs piloted by this MechWarrior gain ALL THE EVASIVE CHARGES after moving (can exceed the unit's maximum). If the move is not a sprint, jump, or charge to melee, the 'Mech also gains ENTRENCHED (50% stability damage reduction).",
    "Icon" : ""
}
```

## StatisticEffectData Extensions

Abilifier now ships with an additional config .json called `EffectDataExtensions.json`. This file defines optional new capabilities and restrictions for EffectData affecting StatCollections in game. The config comprises a Dictionary, where the "key" is equal to the EffectData.Description.Id for which you want to impose additional restrictions.

**Formatting change in 1.4.0.0**

For example, in `EffectDataExtensions.json` I have:

```
{
	"StatusEffect-PotatoAccuracy": {
		"TargetCollectionsForSearch": {
			"Component": {
				"MustMatchAll": true,
				"TargetCollectionTagMatch": [
					"test_has_tag",
					"test_has_also_tag"
				],
				"TargetCollectionNotMatch": [
					"test_has_not_tag1",
					"test_has_not_tag2"
				]
			},
			"Unit": {
				"MustMatchAll": false,
				"TargetCollectionTagMatch": [
					"test_unit_tag1",
					"test_unit_tag2"
				],
				"TargetCollectionNotMatch": [
					"test_unit_not_tag1",
					"test_unit_not_tag2"
				]
			}
		},
		"TargetComponentTagMatch": [
			"Potato_Gun",
			"Tomato_Gun"
		],
		"MustMatchAllComponent": false
	},
	"AccMod1": {
		"TargetCollectionsForSearch": {
			"Component": {
				"MustMatchAll": true,
				"TargetCollectionTagMatch": [
					"test_has_tag",
					"test_has_also_tag"
				],
				"TargetCollectionNotMatch": [
					"test_has_not_tag1",
					"test_has_not_tag2"
				]
			},
			"Unit": {
				"MustMatchAll": false,
				"TargetCollectionTagMatch": [
					"test_unit_tag1",
					"test_unit_tag2"
				],
				"TargetCollectionNotMatch": [
					"test_unit_not_tag1",
					"test_unit_not_tag2"
				]
			}
		},
		"TargetComponentTagMatch": [
			"Potato_Gun",
			"Tomato_Gun"
		],
		"MustMatchAllComponent": false
	}
}
```

On its own this will do nothing, but lets say I have an AbilityDef that contains EffectData of the following:

```
{
	"durationData": {
		"duration": 1,
		"stackLimit": 1
	},
	"targetingData": {
		"effectTriggerType": "OnActivation",
		"effectTargetType": "Creator",
		"showInStatusPanel": true
	},
	"effectType": "StatisticEffect",
	"Description": {
		"Id": "StatusEffect-PotatoAccuracy",
		"Name": "Overcharged Targeting",
		"Details": "+20 Accuracy",
		"Icon": "uixSvgIcon_PotatoGun"
	},
	"nature": "Buff",
	"statisticData": {
		"abilifierId": "AccMod1", // if this defined, Description.Id will be ignored even if it exists in EffectDataExtensions!
		"statName": "AccuracyModifier",
		"operation": "Float_Add",
		"modValue": "-20",
		"modType": "System.Single",
		"additionalRules": "NotSet",
		"targetCollection": "Weapon",
		"targetWeaponCategory": "NotSet",
		"targetWeaponType": "NotSet",
		"targetAmmoCategory": "NotSet",
		"targetWeaponSubType": "NotSet"
	}
}
```

**Important** With v1.2.0.3, StatisticEffectData blocks now have new field `abilifierId`. If defined, StatisticEffectExtensions will search for this field ID key rather than `EffectData.Description.Id`. This allows more "compact" StatisticEffectExtensions settings for effects that use the same matching logic but need different `EffectData.Description.Id`s for stackLimit purposes. If `abilifierId` is defined, Description.Id will be ignored even if it exists in EffectDataExtensions!

Normally, the above ability would give +20 accuracy to <i>all</i> weapons on the affected unit. But because `abilifierId` "AccMod1" or (if abilifierId was not defined) `"Id": "StatusEffect-PotatoAccuracy",` from the abilitydef matches one of those keys from EffectDataExtensions (either "StatusEffect-PotatoAccuracy" or "AccMod1" in this case), the EffectDataExtensions process applies.

This is essentially a two-step process. Step one uses the `TargetCollectionsForSearch` (**note the rename in v1.4.0.0!**) to determine if the given unit is eligible for the effect to apply. Step two uses `TargetComponentTagMatch` to determine _what specific things_ on the unit the effect will apply to (generally specific weapons or components).

`TargetCollectionsForSearch` is a dictionary which can have entries with keys `NotSet`, `Component`, `Unit`, and/or `Pilot`. Using key `NotSet` will disable this first check. If multiple collections are configured as in the above example, all collections must contain a match for the effect to apply. `MustMatchAll` determines whether _all_ configured tags must be matched, or if _any_ single match is sufficient for a given collection. `TargetCollectionMatch` lists tags which must be present, while `TargetCollectionNotMatch` lists tags which must not be present. **NOTE**: Leaving `TargetCollectionsForSearch` null or empty, or using the `NotSet` key skips this first step of filtering, and will proceed directly to processing `TargetComponentTagMatch`. 

In the above example config for "StatusEffect-PotatoAccuracy", `TargetCollectionsForSearch` contains `Component`, with `MustMatchAll` is set to true. For a potential unit to meet the requirements, it must contain components
 with `test_has_tag` AND `test_has_also_tag`, and it MUST NOT contain components with `test_has_not_tag1` AND `test_has_not_tag2`. In addition, `TargetCollectionsForSearch` contains `Unit`, but with `MustMatchAll` set false. For a potential unit to meet the requirements, its MechDef or VehicleDef tags must contain either `test_unit_tag1`, OR `test_unit_tag2`. It likewise cannot have either of `test_unit_not_tag1` or `test_unit_not_tag2`. Again, requirements for both configured collections, Component and Unit, must be met.

Ok, so lets say all those requirements are met. The 2nd step filters using `TargetComponentTagMatch`. In this case, we have `Potato_Gun` amd `Tomato_Gun`, and `MustMatchAllComponent` is set to false. This means that components containing either the `Potato_Gun` or `Tomato_Gun` tags will receive the +20 accuracy bonus. Similar to above, if `MustMatchAllComponent` was set to true, only components containing _both_ `Potato_Gun` AND `Tomato_Gun` will receive the effect. If `TargetComponentTagMatch` is left empty, all valid components (see below note) would receive the effect.

Note: this function still respects the existing restrictions of targetCollection, targetAmmoCategory, etc., with a few differences based on what TargetCollection, if any, is selected. For example, the above is using `"targetCollection": "Weapon",`, so only Weapons are available to be used here, and are just filtered further by the TargetComponentTagMatch match requirement.

If targetCollection = NotSet (or missing from the EffectData), TargetComponentTagMatch searches the unit tags of the target if it is a valid AbstractActor. So if it is a Mech, it will search for a matching MechDef tag, Vehicle: VehicleDef Tag, and Turret: TurretDef Tag.

If targetCollection = Pilot, TargetComponentTagMatch searches the pilotTags of the targets pilot.

## Additional AbilityDef Fields

A few new fields have been added to AbilityDefs, providing some additional functionality.

### CBill Costs

An optional new field in AbilityDefs "CBillCost" defines a per-use cbill cost for using the ability. Does not need to be paired with ResolveCost or use the Resolverator module as discussed below.

### Ability Single Targeting Fix

An optional new field in AbilityDefs "TargetFriendlyUnit" allows abilities with `"Targeting": "ActorTarget",` and `"effectTargetType": "SingleTarget",` to target friendly units, enemies, or both. 

`"TargetFriendlyUnit": "BOTH",` - ability can target both Friendly and Enemy units

`"TargetFriendlyUnit": "FRIENDLY",` - ability can target only Friendly units

`"TargetFriendlyUnit": "ENEMY",` - ability can target only Enemy units

Does not need to be paired with ResolveCost or use the Resolverator module as discussed below.

### Universal Cooldown Tweaks

An optional new field in AbilityDefs "TriggersUniversalCooldown" determines whether an ability triggers the vanilla "universal cooldown" when activated. Vanilla behavior = true, where activating an ability makes all other abilities unavailable for that activation. Adding this field and setting it to false in the AbilityDef will prevent that ability from triggering the universal cooldown for all other abilities.

Additionally, an optional new field in AbilityDefs "IgnoresUniversalCooldown" determines whether <i>that specific ability</i> is subject to the universal cooldown. Adding this field and setting it to true in the AbilityDef will prevent this ability from being subject to the universal cooldown.

#### Start in Cooldown

Abilities with an optional new field "StartInCooldown" set to true will start the contract in their cooldown period.

#### Ability use restricted by tags

Abilities with an optional new array of string "RestrictedTags" will be unable to be used if the unit with the ability has any of the defined tags. Also applies to pilot tags. In addition, ability description text can now accept either `[RestrictedTags]` or `{11}` to parse the description tags into the description so you don't have to manually type them. 

## New Module - Resolverator!
**This module depends on CustomActivateableEquipment**

If enabled, this module tracks resolve separately per-pilot rather than as a team. In addition, <i>regular</i> abilities can now have a resolve cost associated with them. Abilities' resolve cost is dictated by adding a "ResolveCost" field to the AbilityDef:
eg
```
{
    "Description": {},
    "DisplayParams": "ShowInMWTRay",
    "ReqSkill": "Gunnery",
    "ReqSkillLevel": 5,
    "ActivationTime": "ConsumedByFiring",
    "ActivationCooldown": 4,
    "Targeting": "ActorSelf",
    "ResolveCost": 25,
    "CBillCost": 5000
```

resolveGenBaseMult, resolveCostBaseMult, and resolveRoundBaseMod are all (float) actor stats which can be modified by skills, equipment, etc. In addition, `maxResolveMod` is an actor stat which is added to `CombatGameConstants.MoraleConstants.MoraleMax` to define the maximum resolve the actor can have. Also (obviously) modifiable by equipment, skills, etc.

Tracking resolve costs per-pilot means a hefty rebalance of resolve generation will likely be needed. In addition to the values in CombatGameConstants under `"MoraleConstants": {`, other values that may need changing are:

**UPDATE FOR v1.1.6.0**

Abilifier now supports dynamic resolve costs for ability tooltips and descriptions. For the descriptors listed below (found in CombatGameConstants.CombatUIConstants), the `{0}` or [ResolveCost] args will autoreplace with the corresponding values from `CombatGameConstants.MoraleConstants` _and_ will reflect individual unit values for the unit statistic `resolveCostBaseMult`. In addition, the `{10)` can be used in the `Description.Details` field of AbilityDefs to give the value of the `ResolveCost` set for that ability. The final displayed value will also reflect changes due to `resolveCostBaseMult`. E.g.

```
{
	"Description": {
		"Id": "AbilityDefG5a",
		"Name": "BATTLELORD",
		"Details": "ACTION: Supercharge your mech for a turn, dealing 15% greater damage and hitting with +2 accuracy for the turn. Generates an extra 30 heat this turn. 3 turn cooldown. <b><color=#099ff2>Costs {10} Resolve to use!</color></b>",
		"Icon": "uixSvgIcon_skullAtlas"
	},
```

```
    "MoraleCostAttackDescription": {
      "Name": "PRECISION STRIKE COST",
      "Details": "Cost: {0} Resolve"
    },
    "MoraleCostAttackDescriptionLow": {
      "Name": "PRECISION STRIKE COST LOW",
      "Details": "Cost: <color=#F04228FF>{0} Resolve (this MechWarrior has Low Spirits)</color>"
    },
    "MoraleCostAttackDescriptionHigh": {
      "Name": "PRECISION STRIKE COST HIGH",
      "Details": "Cost: <color=#85DBF6FF>{0} Resolve (this MechWarrior has High Spirits)</color>"
    },
    "MoraleCostDefendDescription": {
      "Name": "VIGILANCE COST",
      "Details": "Cost: {0} Resolve"
    },
    "MoraleCostDefendDescriptionLow": {
      "Name": "VIGILANCE COST LOW",
      "Details": "Cost: <color=#F04228FF>{0} Resolve (this MechWarrior has Low Spirits)</color>"
    },
    "MoraleCostDefendDescriptionHigh": {
      "Name": "VIGILANCE COST HIGH",
      "Details": "Cost: <color=#85DBF6FF>{0} Resolve (this MechWarrior has High Spirits)</color>"
    },
```

## Settings

Settings available in the mod.json:
```
  "Settings": {
  	"debugExpiration": false,
	"enableTrace": false,
	"enableLog": false,
	"enableResolverator": true,
	"disableResolveAttackGround": true,
	"resolveGenBaseMult": 1.0,
	"resolveCostBaseMult": 1.0,
	"resolveCostHighMult": 1.0,
	"resolveCostLowMult": 1.0,
	"usePopUpsForAbilityDesc": false,
	"debugXP": false,
	"extraFirstTierAbilities": 0,
	"extraAbilities": 1,
	"extraAbilitiesAllowedPerSkill": 1,
	"cleanUpCombatUI": true,
	"nonTreeAbilities": 1,
	"skillLockThreshold":8,
	"extraPreCapStoneAbilities":0,
	"usingCACabilitySelector":false,
	"usingHumanResources": false,
	"disableCalledShotExploit": false,
	"proceduralTagCleanup": {
			"pilot_nomech_crew": [
				"pilot_mechwarrior"
			]
		},
	"tagTraitForTree": 
		{	
			"pilot_vehicle_crew": "TraitDefIAmTank",
			"pilot_mechwarrior": "TraitDefIAmMech",
			"pilot_defaultTree": "TraitDefIAmMech"
		},
	"defaultTagTraitForTree": {"pilot_defaultTree": "TraitDefIAmMech"},
	"defaultTagTraitException": "pilot_nomech_crew",
	"abilityReqs":
		{
		"AbilityDefG5a":["AbilityDefG8a","AbilityDefG8b"],
		"AbilityDefGu5a":["AbilityDefGu8a"]
		}
	},
	"retrainerSettings": {
			"enableRetrainer": true,
			"cost": 500000,
			"onceOnly": false,
			"trainingModuleRequired": true,
			"ignoredAbilities": [
				"AbilityDefBDInit",
				"AbilityDefBlueAP",
				"AbilityDefLokiGhost",
				"AbilityDefWulfSRM",
				"AbilityDefTexHealth",
				"AbilityDeftboneHunt",
				"AbilityDefTurtrusEnergy",
				"AbilityDefRevostaeIndirect",
				"AbilityDefShadeMelee",
				"AbilityDefGrampaTex",
				"AbilityDefCMDHunter",
				"AbilityDefCMDOperative",
				"AbilityDefCMDPilot",
				"AbilityDefCMDRacer",
				"AbilityDefCMDSmuggler",
				"AbilityDefCMDTrainee"
			],
			"confirmAbilityText": "Confirming this Ability selection is semi-permanent. You may only have two Level 5, one Level 8, and one level 10 Abilities. With the Training Module 2 argo upgrade, Mechwarriors can be retrained for 500,000 c-bills by shift-clicking on the Pilot's skills tab."
		}
```

`debugExpiration` - **this setting defaults to true!** unless you are explicitly working with me to debug effect duration, you should set this to false

`enableTrace` and `enableLog` (bools) allow logging.

`enableResolverator` - bool, enables pilot resolve overhaul module

`disableResolveAttackGround` - bool, enabled by default, only used in CAE/CAC build. disable resolve processing for all "attack ground" attacks. also disable resolve bonus/malus for % shots hit/missed for weapons that proc AOE damage or mines.

~~`resolveGenTacticsMult` - float, multiplier modifies resolve generation according to tactics skill. Initialized as an actor stat, which can be modified be equipment, abilities, etc.~~ deprecated. just alter resolveGenBaseMult using traits.

~~`resolveCostTacticsMult` - float, multiplier modifies resolve costs of abilities according to tactics skill. Initialized as an actor stat, which can be modified be equipment, abilities, etc.~~ deprecated. just alter resolveCostBaseMult using traits.

`resolveGenBaseMult` - float, base multiplier for all resolve generation. Initialized as an actor stat, which can be modified be equipment, abilities, etc.

`resolveCostBaseMult` - float, base multiplier for all resolve costs. Initialized as an actor stat, which can be modified be equipment, abilities, etc.

`resolveCostHighMult` - float, multiplier for resolve costs if pilot has high spirits. multiplied directly onto `resolveCostBaseMult`. NOTE: DefensivePushHighMoraleCost, OffensivePushLowMoraleCost, etc from MoraleConstants have been deprecated! Resolve cost of Vigilance/DefensivePush will be MoraleConstants.DefensivePushCost x resolveCostBaseMult x resolveCostHighMult or resolveCostLowMult as appropriate! Likewise for CalledShot/OffensivePush! Similarly, `MoraleCostDefendDescriptionLow`, etc in CombatUIConstants should be updated so that their listed resolve cost uses the `{0}` or [ResolveCost] arguments so that their resolve cost can be dynamically updated based on `resolveCostBaseMult`.

`resolveCostLowMult` - float, multiplier for resolve costs if pilot has low spirits. multiplied directly onto `resolveCostBaseMult`. NOTE: DefensivePushHighMoraleCost, OffensivePushLowMoraleCost, etc from MoraleConstants have been deprecated! Resolve cost of Vigilance will be baseline MoraleConstants.DefensivePushCost x resolveCostBaseMult x resolveCostHighMult or resolveCostLowMult as appropriate! Likewise for CalledShot/OffensivePush! Similarly, `MoraleCostDefendDescriptionLow`, etc in CombatUIConstants should be updated so that their listed resolve cost uses the `{0}` or [ResolveCost] arguments so that their resolve cost can be dynamically updated based on `resolveCostBaseMult`.

**NOTE** the unit statistic (float) `resolveRoundBaseMod` can be used to add a flat resolve gain each round *specific to the unit* (as opposed to vanilla statistic `MoraleBonusGain` which will still add bonus resolve to each unit on the team). Keep in mind this "flat" bonus will still be affected by `resolveGenBaseMult`, however.

`usePopUpsForAbilityDesc` bool, sets Abilifier to use hover tooltips for Ability descriptions as described above.

`debugXP` bool, grants 100000 XP when XP is spent (useful for testing newly added abilities).

`extraFirstTierAbilities` int, allows players to take additional 1st tier abilities within the limit of total abilities.

`extraAbilities` int, allows players to take additional (>3) abilities. Theoretically only limited by screen space. Individual buttons get closer to eachother as more abilites are added, but usable with 5 additional abilities. Image below is with 5 additional abilities and `cleanUpCombatUI` set to true.

![HoverPop](https://github.com/BattletechModders/Abilifier/blob/master/doc/alltheabilities.png)

`extraAbilitiesAllowedPerSkill` int, allows players to take additional (>2) abilities within a given skill area. <b>Assumes developer has included new abilities at skill levels other than 5 and 8.</b> Only 1 ability per-skill-level may be taken, however.

`cleanUpCombatUI` bool, if `true`, removes decorative chevrons and vertical bars from the ability tray in combat. Recommend setting to `true` unless NOT enabling CAC Attack Ground button AND `extraAbilities` <=1.

`nonTreeAbilities` int; adds extra button slots in combat UI for abilities to be added to PilotDefs <i>independent</i> of the ability tree. Think Victoria's Fire and Steel ability, that sort of thing. Abilities added to pilots in this way <b>must not</b> have `IsPrimaryAbility`, `ReqSkill`, or `ReqSkillLevel` in their AbilityDefs. Likewise, they should <b>not</b> be added to the Progression section of SimGameConstants. They <b>do</b>, however need `"DisplayParams" : "ShowInMWTRay",` if you want to be able to see them in the ability tray in missions.

`skillLockThreshold` int, defines a threshold level for skills; once a pilot has reached that skill level, they can *only* take abilities above that level in that skill. Intended for use with additional lvl 10 abilities; in the settings above, once a pilot reaches lvl 8 (2nd tier ability by default), the only abilities at or above lvl 8 that they can take must be in that skill. However, pilots can always take abilities *below* the threshold (within the constraints of the above settings). Set to 10 (effectively nonfunctional) by default.

To clarify: a pilot could, for example, have abilities at the following levels:
```
Gunnery: 5, 8, 10
Guts: 5
```
but you could not have 
```
Gunnery: 5, 8
Guts: 5, 8
```

`extraPreCapStoneAbilities` int. Decides additional number of skills allowed to give abilities before the CapStone ability (works in concert with skillLockThreshold).

`usingCACabilitySelector` bool. if true, makes Abilifier compatible with newer versions of CustomBundle/CAC that incorporate an "ability selector" for the combat UI. If false, Abilifer will create `extraAbilities + nonTreeAbilities` traditional ability button slots.

`usingHumanResources` bool. if true, Abilifier will let HR handle adding tags for below.

`disableCalledShotExploit` - new setting, optionally disabled "called shot exploit" where a player can activate called shot, back out, activate another ability, and then use called shot regardless of available resolve. downside is that in order to legitimately back out of called shot and get the resolve back, player must hit Esc a few times.

`proceduralTagCleanup` - Dictionary <string, List<string>> - new setting, can remove certain pilot tags from procedurally generated pilots if they have other specified tags. if the pilot contains a tag with the "key", then all tags in the "value" list will be removed. i.e. below, any pilot with `pilot_nomech_crew` will have `pilot_mechwarrior` removed.
```
"proceduralTagCleanup": {
			"pilot_nomech_crew": [
				"pilot_mechwarrior"
			]
		},
```

`tagTraitForTree` dictionary<string, string> - further supports ability "tree" restrictions for procedurally generated piots. the "key" in this case is a Pilot Tag (primarily added by Human Resources mod. If the pilot in question has the "key", then the ability/trait ID indicated in the "value" is given to the pilot. The intent is for this "trait" to be the required prereq for subsequent abilities in the "tree". This ability/trait can simply be a "dummy" trait as in the following:
```
{
	"Description" : {
		"Id" : "TraitDefVehicleTraining",
		"Name" : "",
		"Details" : "",
		"Icon" : ""
	},
	"ShortDesc" : "",
    "DisplayParams" : "NeverShow",
	"ActivationTime" : "Passive",
	"EffectData" : []
}
```

`defaultTagTraitForTree`: dictionary, string, string - ALL pilots will recieve this tag and trait unless they have the tag defined in `defaultTagTraitException`

`defaultTagTraitException`: see above

`abilityReqs` dictionary, strings. new in 1.05, allows devs to set up true ability "trees", where the 1st ability (dictionary key) is required for the player to take any of the subsequently listed abilities (dictionary value, list of strings). For example, in the above settings, a player can <i>only</i> take `AbilityDefG8a` or `AbilityDefG8b` if they had previously taken `AbilityDefG5a`; all other abilities for this level will be available.

Unavailable abilities will still be shown in the ability-chooser dialogue box, with text added indicating what missing ability is required, but they will not be selectable:

![HoverPop](https://github.com/BattletechModders/Abilifier/blob/master/doc/abilityReqs.png)


`ticksOnMovementDistanceIDs`: effect data with IDs in this list and which have `ticksOnMovements` set true in durationData will use their target units last movement distance (`unit.DistMovedThisRound` value) to decrement duration rather than simply decrementing by one if they moved. Note that because units' `DistMovedThisRound` includes distance spent going uphill, around obstacles, etc, it will rarely be cleanly divisible by the games nominal hex size. E.g. If game hex size is set to 28, and an effectData has duration set to 56, that will not always equate to "two hexes" of movement.

## Retrainer Module

v1.2.0.0 and higher have integrated Retrainer; standalone Retrainer mod is no longer necessary. Includes kmission's fancy retrain button in barracks, eliminating need to shift-click.

```
"retrainerSettings": {
			"enableRetrainer": true,
			"cost": 500000,
			"onceOnly": false,
			"trainingModuleRequired": true,
			"ignoredAbilities": [],
			"confirmAbilityText": ""
		}
```

`enableRetrainer`, bool. if true, will use integrated retrainer module. obviously not compatible with standalone Retrainer

`cost`, int: cbill cost to retrain

`onceOnly`, bool: whether you can only retrain a given pilot once

`trainingModuleRequired`, bool: whether the argo training module 2 upgrade is required to retrain

`ignoredAbilities`, List [strings]: list of AbilityDef names which will <i>not</i> be removed when retraining. Intended to be used with pre-set, non-tree abilities (if you put a normal ability here, you'd keep the ability but still get refunded the XP when retraininer).

`confirmAbilityText`: string. if not empty, will replace in-game popup text when confirming abilities (mostly for use if Abilifier is allowing more than the default number of abilities) 

## AbilityRealizer

As of v1.3.0.0, Abilifier has absorbed the AbilityRealizer mod. Settings for AbilityRealizer module can be found in `AbilityRealizerSettings.json` in the Abilifier mod folder. All existing settings and behavior have been maintained.

Settings clarification:

Provide support for modding the ability tree and abilities without requiring modders to completely redo all of the PilotDefs, as well as providing a mechanism for updating pilots that are already stored in saves.

`DumpAbilityDefNamesAtAddToTeam` - bool, if true all pilot AbilityDefNames will be dumped to log when added to teams for debugging purposes

`AddTreeAbilities` - bool, if true pilots will be given abilities as appropriate according to the ability tree in simgameconstants. doesnt play nicely with core Abilifier functionality of allowing multiple ability options, and may result in pilots being given moore abilities than they can legally have. No, I'm not updating it to work.

`RemoveNonTreeAbilities` - bool, if true pilots will have abilities removed which are not present in the SGC ability tree. Really only useful if you change/deprecate an abilitydef ID, but can result in pilots losing unique abilities that are not in the ability tree.

`IgnoreAbilities` - list<string> - list of abilitydef IDs which are ignored by above Add/Remove settings. 

Original readme for AbilityRealizer follows:

### What It Currently Does

* Keeps all pilots/pilot defs up-to-date with the current state of the ability tree (stored in SimGameConstants)

* Prevent crashes/save game loss from changing the ability tree

* Changes the barracks UI to show tooltips for passive abilities that are not primary abilities

* Can add abilities based on Faction or Tag

* Can swap abilities for the AI (until adding to the AI is added)

### Ignoring Pilots By Tag

Pilots that have any of these tags will be ignored

```json
"IgnorePilotsWithTags": [ "pilot_release_skirmish", "pilot_release_ksbeta" ]
```

### Adding Abilities based on Faction/Tag

Add to the `FactionAbilities` or `TagAbilities` in the settings

```json
"FactionAbilities": {
    "AuriganPirates": [ "MyAbilityDef1", "MyAbilityDef2" ]
},
```

```json
"TagAbilities": {
    "commander_career_soldier": [ "MyAbilityDef3", "MyAbilityDef4" ]
},
```

### Swapping AI Abilities

Add to the `SwapAIAbilities` in the settings

```json
"SwapAIAbilities": {
    "AbilityDefG8": "AbilityDefG8AI"
},
```

### Upgrading Abilities

The setting "UpgradeAbilities" takes an array of "UpgradeAbiility" values which must have the following defined:

"Prereq" - AbilityDef ID of the prerequisite ability which will be replaced
"Skill" - skill type, e.g. Guts
"Level" - level of the skill, e.g. 7
"Upgrade" - AbilityDef ID of the "upgraded" ability that will replace the "Prereq" ability.

e.g.:
```
"UpgradeAbilities": [
			{
				"Prereq": "AbilityDef_CarefulManeuvers",
				"Skill": "Piloting",
				"Level": 5,
				"Upgrade": "AbilityDef_LessCarefulMoreManeuvers"
			}
		]
```
