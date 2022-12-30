# **A Guide to Surface Rules**
### **(Minecraft Java Edition 1.18.2)**

*(Disclaimer: I am not an expert with surface rules. It’s very possible some things here are wrong. If anyone has corrections/suggestions feel free to reach out. In particular, I did not go to the effort to manually test every example, so if you decide to try one out and it doesn't validate or generate correctly, please let me know.)*

## **What are surface rules?**

Under 1.18+ world generation, surface builders no longer exist, and their role has been replaced with surface rules. Surface rules exist within the noise_settings file, and as such apply dimension-wide. What surface rules do is take a block of stone (or whatever your `default_block` within the same file is set to; the rest of this guide is written as if stone is the `default_block`) and replace it with a different block depending on what the rules say. In vanilla Minecraft, surface rules are responsible for such things as: placing grass at the surface of the world, placing sand at the surface of desert biomes, creating the different colored bands of terracotta within badlands biomes, creating the deepslate layer, making the bedrock floor, making sure grass blocks don't appear underwater, and more. There is a lot of utility to them.

Individually, surface rules (sometimes) aren't *too* complicated, with some even being quite simple, but when used together they can be extremely powerful tools. As an example, Ganar uses surface rules to create custom noise caves and ore veins (like the long veins of copper and iron introduced in 1.18)— currently this is the only way known to do this without modding.

## **When do surface rules run?**

Surface rules take effect after: the shaping of the terrain (so hills, mountains, ocean basins, etc. will all be in their correct shapes); the filling of the oceans, rivers, and aquifers with water and lava, and the filling of the lava table; the generation of noise and noodle caves; **and the generation of the large ore veins introduced in 1.18.** Biomes have already been laid out too (although they’re functionally identical before surface rules are applied), and all the perlin noises have been sampled. By the time surface rules start being applied across the world, the only blocks are air and cave air, stone (or your default block if it's different), water, lava, and the blocks that make up the new large copper and iron veins.

Surface rules come before everything else not listed above, so no trees yet, no villages, mineshafts, or other structures yet, **and no cave carvers yet.** (This last one is important for the `minecraft:stone_depth` condition; I’ll say more when I talk about that in detail.)

## **How do I format surface rules?**

Each noise_settings file actually only comes with one surface rule, but there’s an easy way to nest surface rules within each other, which is how we get to have more. But essentially, within the root tag of your noise settings file, it should look like this:
```json
"surface_rule": {
	…
}
```
Yep, that’s it. Everything else goes in that `…` You can place any surface rule here, whether that’s a block, a condition, or a sequence. I’ll explain all of these next, but you probably want your root surface rule to be a `sequence`.

**Do make sure that all instances of `"` ARE actually `"`. Some text editors will autocorrect them to something like `“` or `”`, which will NOT work!**

(Whenever there's multiple fields within a set of braces, they can be listed in any order. For most of this document, I order them in the way that makes sense to me.)

### **Also, before we get started, this guide is meant to be read in order. If you need brushing up on something (as I did many times while writing), of course feel free to skip to whatever section you need, but I would recommend reading this document top to bottom (perhaps across multiple sittings…) if you're new to surface rules. That said, examples are presented in case you'd like them; you do not have to read all the examples here to understand surface rules.**

*(Click on the arrow within each section to see details, formatting, and examples with explanations.)*

***

***

***





## **Surface Rule - block**

<details>
  <summary>Used to place a block.</summary>
  <br>

This surface rule places the specified block after preceding conditions have been met. If there are no preceding conditions, the specified block replaces every single stone block available.

  <h3>Formatting:</h3>
  
```json
{
	"type": "minecraft:block",
	"result_state": {
		"Name": <block>,
		"Properties": {
			<property>: <value>,
			<property>: <value>,
			…
		}
	}
}
```
where `<block>` is the namespaced id of the block to place,
`<property>` is the name of a block state applying to that block,
and `<value>` is the value you want that block state set to.

When the block specified has no block states, you can omit the entire `"Properties": {…}` section.
`<value>` *must* be put in quotations even when it's an integer value.

> If you are unsure of what properties a block should have, you can search up the block on the Minecraft wiki and it should list all of the properties and their possible values underneath the "Data values" section, "Block states" subsection. (If the "Block states" subsection is missing, that means the block has no properties. If the "Data values" section is missing, that means the Minecraft wiki doesn't have this information.)

> Alternatively, you can look at the block in-game in F3 mode (which might be especially helpful if you're working with modded blocks), and it should list all the block's current properties on the right hand side in the "Targeted Block:" section, in between the block's name and the tags it's a part of (which always begin with a `#`). However, this will only tell you the values of the block you're currently looking at, not all possible values for each property.

  <h3>Examples:</h3>
  
  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:block",
	"result_state": {
		"Name": "minecraft:dirt"
	}
}
```
> ^ This replaces all valid blocks of stone with blocks of dirt. Dirt has no properties, so that field has been omitted.
    
  </details>
  
  <details>
    <summary>Example 2</summary>
  
```json
{
	"result_state": {
		"Name": "minecraft:grass_block",
		"Properties": {
			"snowy": "false"
		}
	},
	"type": "minecraft:block",
}
```
> ^ This replaces all valid blocks of stone with grass blocks. `snowy` being set to "false" ensures that the grass block has its normal texture. Setting `"snowy"` to `"true"` would instead place the texture with white grass that grass blocks have when snow layers are on top of them (the snow layers themselves would not be placed).

> Note that `"type": "minecraft:block"` and `"result_state": {…}` have switched places. For all surface rules, it doesn't matter what order fields are listed in— so long as all of the necessary fields are included, and so long as commas are in the right places, everything will work.
    
  </details>
  
  <details>
    <summary>Example 3</summary>
  
```json
{
	"type": "minecraft:block",
	"result_state": {
		"Properties": {
			"east": "true",
			"north": "true",
			"west": "false",
			"south": "true",
			"waterlogged": "false"
		},
		"Name": "oak_fence"
	}
}
```
> ^ This replaces all valid blocks of stone with oak fences, which reach out as if there were solid blocks on their east, north, and south faces, but not on their west, and these fences are not waterlogged. Note again that fields can be listed in any order. *(For the rest of this document, I'll be using the order that makes sense to me.)*
    
  </details>
    
</details>

***
  
***
  
***
  
  



## **Surface Rule — condition**

<details>
  <summary>Used to run a surface rule under a certain condition.</summary>
  <br>
  
This surface rule runs checks whether or not a requirement is met at any given location. If the requirement is met, it runs the surface rule within its `"then_run"` field. If the requirement is not met, nothing happens and we continue to the next surface rule. Condition surface rules can be nested inside each other to check for multiple requirements before placing a block.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		<condition>
	},
	"then_run": {
		<surface rule>
	}
}
```
where `<condition>` is a requirement (usually multiple lines) that must be met to continue,
and `<surface rule>` is the surface rule (always multiple lines) that runs on success.

Possible surface rules are limited to `block`, `condition`, and `sequence`. Possible conditions are `biome`, `y_above`, `water`, `vertical_gradient`, `stone_depth`, `hole`, `above_preliminary_surface`, `noise_threshold`, `temperature`, `steep`, and `not`. For an explanation of what each of these conditions does, see below.

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>
        
```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:dripstone_caves"
		]
	},
	"then_run": {
		"type": "minecraft:block",
		"result_state": {
			"Name": "minecraft:dripstone_block"
		}
	}
}
```
> ^ This replaces all stone with dripstone blocks, but only inside the dripstone caves biome. Note that the `"then_run": {…}` section is just another block surface rule.
        
  </details>
      
  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:y_above",
		"anchor": {
			"absolute": 60
		},
		"surface_depth_multiplier": 0,
		"add_stone_depth": false
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:biome",
			"biome_is": [
				"minecraft:desert"
			]
		},
		"then_run": {
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:sand"
			}
		}
	}
}
```
> ^ This uses nested conditions to replace all stone above y=60 in the desert biome with sand. In all other biomes there will still be stone, and at y=60 and below in desert biomes there will still be stone.
      
> In this example, you could switch the `"if_true": {…}` sections and it would have the same effect. That won't be the case once sequences are involved, but any time you have multiple conditions in a row without any sequences in between, the order doesn't matter.
        
  </details>
      
</details>
      
***
      
***
      
***
      
      



## **Surface Rule — sequence**

<details>
  <summary>Used to run multiple surface rules in general and under shared conditions.</summary>
  <br>

This surface rule is really just a list of other surface rules. The rules within it run in order (from first to last / top to bottom). Once each block meets the conditions of one of the surface rules listed, it places the block and doesn't check the others. Blocks that don't meet the conditions of the first surface rule listed are then checked against the second; if they don't meet the conditions of the second, they are then checked against the third; and so on. Note that you can set a surface rule in this sequence to place stone (or whatever your default block it), and it will not be replaced by other surface rules that come after it, both within its sequence and throughout the rest of the document.

  <h3>Formatting:</h3>
  
```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			<surface rule>
		},
		…,
		{
			<surface rule>
		}
	]
}
```
where `<surface rule>` is a surface rule to run (can be a block, condition, or another sequence).

You can have any number of surface rules within a sequence, but remember to include commas between them, and to have no comma after the last one.

  <h3>Examples:</h3>
  
  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:biome",
				"biome_is": [
					"minecraft:dripstone_caves"
				]
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:dripstone_block"
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:andesite"
			}
		}
	]
}
```
> ^ This sequence places dripstone blocks in the dripstone caves biome, and andesite everywhere else.
    
  </details>
  
  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:biome",
				"biome_is": [
					"minecraft:dripstone_caves"
				]
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:dripstone_block"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:biome",
				"biome_is": [
					"minecraft:lush_caves"
				]
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:moss_block"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:y_above",
				"anchor": {
					"absolute": 50
				}
				"surface_depth_multiplier": 0,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:diorite"
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:andesite"
			}
		}
	]
}
```
> ^ This sequence places dripstone blocks in the dripstone caves biome, moss blocks in the lush caves biome, diorite above y=50 in all other biomes, and andesite at and below y=50 in all other biomes. Note that the dripstone blocks and moss blocks are *not* limited by height, because the `y_above` condition only applies to the surface rule placing diorite.
    
  </details>
  
  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:y_above",
		"anchor": {
			"absolute": 32
		},
		"surface_depth_multiplier": 0,
		"add_stone_depth": false
	},
	"then_run": {
		"type": "minecraft:sequence"
		"sequence": [
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:biome",
					"biome_is": [
						"minecraft:dripstone_caves"
					]
				},
				"then_run": {
					"type": "minecraft:sequence",
					"sequence": [
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:y_above",
								"anchor": {
									"absolute": 50
								},
								"surface_depth_multiplier": 0,
								"add_stone_depth": false
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:dripstone_block"
								}
							}
						},
						{
							"type": "minecraft:block"
							"result_state": {
								"Name": "minecraft:granite"
							}
						}
					]
				}
			},
			{
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:andesite"
				}
			}
		]
	}
}
```
> ^ As seen here, sequences can be nested with each other and with condition surface rules. When a block meets a sequence's conditions, but none of the requirements for the surface rules within that sequence, it goes "outwards one level" (I can't think of a better way to describe this, sorry). If a block is left unfulfilled by all surface rules within the whole document, it remains as stone (the default block).

> This sequence places dripstone blocks in the dripstone caves biome above y=50, granite in the dripstone caves biome between y=32 and y=50, and andesite in every other biome above y=32. Because there are no surface rules running on blocks at and below y=32, those remain stone.
    
  </details>

</details>

***

***
        
***





#### *The rest of this document will be used to explain the various conditions that can be referenced in the `is_true` field of `condition` surface rules.*

***

***

***





## **Condition — biome**

<details>
  <summary>Limits surface rules by biome.</summary>
  <br>

This condition tells the game to only apply the surface rule on blocks within any one of the biomes listed.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:biome",
	"biome_is": [
		<biome>,
		…
	]
}
```
where `<biome>` is the namespaced ID of a biome.

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:lush_caves"
		]
	},
	"then_run": {
		"type": "minecraft:block",
		"result_state": {
			"Name": "minecraft:moss_block"
		}
	}
}
```
> ^ This replaces all blocks in the lush caves biome with moss blocks.

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:frozen_peaks",
			"minecraft:jagged_peaks",
			"minecraft:stony_peaks"
		]
	},
	"then_run": {
		"type": "minecraft:block",
		"result_state": {
			"Name": "minecraft:calcite"
		}
	}
}
```
> ^ This replaces all blocks in the frozen peaks, jagged, peaks, and stony peaks biomes with calcite.

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:biome",
				"biome_is": [
					"minecraft:frozen_peaks",
					"minecraft:jagged_peaks",
					"minecraft:stony_peaks",
					"furbyoverworldredux:amethyst_caves",
					"furbyunderworldredux:sheol"
				]
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:calcite"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:biome",
				"biome_is": [
					"furbyoverworldredux:deep_barren_caves",
					"furbyoverworldredux:dark_caves"
				]
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:deepslate"
				}
			}
		}
	]
}
```
> ^ Custom biomes can also be referenced, alone or in addition to vanilla biomes.

  </details>

</details>

***

***

***





## **Condition — y_above**

<details>
  <summary>Limits surface rules by y-level.</summary>
  <br>

This condition generally checks if a block is above (not at) a certain y-level, although applications can change quite a bit depending on how the fields are defined.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:y_above",
	"anchor": {
		<vertical anchor>
	},
	"surface_depth_multiplier": <integer value>,
	"add_stone_depth": <true or false>
}
```
where `<vertical anchor>` is the a vertical anchor (see next paragraph for explanation),
`<integer value>` is simply an integer,
and `<true or false>` is either `true` or `false`.

Vertical anchors appear in multiple places in custom world generation. There are three possible types to use:
• `"absolute": <integer value>` to define a specific y-value
• `"above_bottom": <integer value>` to define a value based on how high it is above the bottom of the world
• `"below_top": <integer value>` to define a value based on how low it is below the top of the world **(note that this is in reference to the build limit, not the world's surface; also note that here higher values move the height down, in contrast with the other two where higher values move the height up)**

`surface_depth_multiplier` adds some value to the y-value, based on the surface noise value at the location. This value is used in a few conditions here, and from my experience I would guess that it's limited to a value of 0-4, varying by location, but usually 2 or 3. This noise-based value is then multiplied by the integer set in the `surface_depth_multiplier` field; so changing `surface_depth_multiplier` from the default `0` should give you some variety in the specified y-value (if you're worried it would look weird to have a uniform shift in the block). The greater the absolute value of the specified integer, the greater the vertical variation (although the horizontal transition is always patchy when the integer is anything other than 0, and the height changes are sheer rather than rounded when the integer is anything other than 0, 1, or -1— take care when using this condition alone to place see-through blocks like air, water, glass, etc.). Positive values move the transition upwards, and negative values move it downwards. Even small values can have large impacts (e.g. setting this to `5` would cause a maximum variation of 20 blocks vertically).

Setting `add_stone_depth` to `true` moves the y-value check to the nearest surface. So if you have a block at y=50, and the surface is at y=64, the game will check whether 64 is above the y-value specified in the surface rule. Note that noise caves (including aquifers), but not cave carvers, are counted as the "surface" here. Setting `add_stone_depth` to `true` will create a consistent column downwards until we run into a noise cave or the bottom of the world. **(Note also that setting `add_stone_depth` to `true` will move the generation down by one block.)**

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true":  {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:frozen_peaks"
			"minecraft:jagged_peaks"
			"minecraft:snowy_plains",
			"minecraft:snowy_taiga",
			"minecraft:ice_spikes",
			"minecraft:snowy_slopes"
		]
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:y_above",
			"anchor": {
				"absolute": 100
			},
			"surface_depth_multiplier": 0,
			"add_stone_depth": false
		},
		"then_run": {
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:snow_block"
			}
		}
	}
}
```
> ^ This replaces stone with snow blocks in the listed biomes at y-level 101 and above.

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true":  {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:frozen_peaks"
			"minecraft:jagged_peaks"
			"minecraft:snowy_plains",
			"minecraft:snowy_taiga",
			"minecraft:ice_spikes",
			"minecraft:snowy_slopes"
		]
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:y_above",
			"anchor": {
				"absolute": 100
			},
			"surface_depth_multiplier": 2,
			"add_stone_depth": false
		},
		"then_run": {
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:snow_block"
			}
		}
	}
}
```
> ^ This does the same thing but adds a bit of variation in height. I still have `add_stone_depth` set to false because I really don't want snow to go down forever.

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:y_above",
				"anchor": {
					"absolute": 0
				},
				"surface_depth_multiplier": 3,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:stone"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:y_above",
				"anchor": {
					"above_bottom": 1
				},
				"surface_depth_multiplier": 3,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:deepslate",
					"Properties": {
						"axis": "y"
					}
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:bedrock"
			}
		}
	]
}
```
> ^ This sequence places stone in the higher parts of the world, then deepslate in the depths, and finally bedrock everywhere else (which is now just those very bottom), with some variation. Note that even though stone is our default block, placing stone in certain locations prevents it from being overwritten by other surface rules. Without this first stone condition, our second depilate condition would affect pretty much the entire world. However, this also means that we can't replace the stone at the top of the world with grass or anything else later, so take care when placing your default block as a protective measure. I have `surface_depth_multiplier` as a positive value so that the generation moves up rather than down (leading to some deepslate above y=0 and multiple layers of bedrock in some places). Note that the variation tends to be in larger "blobs" rather than individual blocks, so this won't get you gradients like you see in vanilla world generation (more on that later).

  </details>

  <details>
    <summary>Example 4</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:y_above",
				"anchor": {
					"absolute": 63
				},
				"surface_depth_multiplier": 3,
				"add_stone_depth": true
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:stone"
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:gravel"
			}
		}
	]
}
```
> ^ In this sequence, I have weathered, stony beach in mind. I want stone a ways a good ways above the shoreline, and gravel around to sea level. My anchor is at y=63, default sea level, but that's fine because my `surface_depth_multiplier` will move the transition area upwards anyways. I have `add_stone_depth` set to true here, because it doesn't make sense to me to have to have gravel directly underneath stone everywhere, and I don't want gravel to show up underneath all the continents.

  </details>

  <details>
    <summary>Example 5</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:y_above",
				"anchor": {
					"absolute": 75
				},
				"surface_depth_multiplier": 0,
				"add_stone_depth": true
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:y_above",
							"anchor": {
								"absolute": 85
							},
							"surface_depth_multiplier": 1,
							"add_stone_depth": false
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:orange_terracotta"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:y_above",
							"anchor": {
								"absolute": 83
							},
							"surface_depth_multiplier": 1,
							"add_stone_depth": false
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:yellow_terracotta"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:y_above",
							"anchor": {
								"absolute": 81
							},
							"surface_depth_multiplier": 1,
							"add_stone_depth": false
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:white_terracotta"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:y_above",
							"anchor": {
								"absolute": 79
							},
							"surface_depth_multiplier": 1,
							"add_stone_depth": false
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:red_terracotta"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:y_above",
							"anchor": {
								"absolute": 77
							},
							"surface_depth_multiplier": 1,
							"add_stone_depth": false
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:purple_terracotta"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:y_above",
							"anchor": {
								"absolute": 75
							},
							"surface_depth_multiplier": 1,
							"add_stone_depth": false
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:brown_terracotta"
							}
						}
					},
					{
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:terracotta"
						}
					}
				]
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:red_sandstone"
			}
		}
	]
}
```
> ^ Here, I have Badlands in mind. I want terracotta to show up, but only where the terrain gets high enough in the "plateau" regions, with red sandstone in the "basins". I don't want terracotta generating above red sandstone though, so I set `add_stone_depth` to `true` in the condition for the terracotta sequence. Within the terracotta sequence itself, I have several layers of different colored terracotta 2 blocks thick. I want these to be uniform horizontally and not extend vertically, so for each of these I have `add_stone_depth` set to `false`. I also want slight variation, so I set `surface_depth_multiplier` to `1`. The layers are still 2 blocks thick almost everywhere, because each condition's anchor is being multiplied by the same noise value (the surface noise).

  </details>

</details>

***

***

***





## **Condition — water**

<details>
  <summary>Limits surface rules by water/lava depth.</summary>
  <br>

This condition basically checks how deep the water is at a certain place, and succeeds when the water is shallow enough, or when there is no water at all. It's useful if you have things like grass blocks or concrete powder, which would change by themselves over time or when updated while underwater— using this condition you can save some time/performance by setting the underwater state (like dirt or concrete) during world generation instead. It's also useful if you use waterlog-able blocks in surface rules, or if you just want a different block to appear underwater or at certain depths.

**Note that this checks for lava as well as water, and any other liquids you may have from mods.**

**Note also that this only checks for liquids naturally generated by sea level, aquifers, etc. It does NOT include liquids placed by earlier surface rules, from surface lava pools, etc.**

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:water",
	"offset": <integer value>,
	"surface_depth_multiplier": <integer value>,
	"add_stone_depth": <true or false>
}
```
where `<integer value>` is an integer,
and `<true or false>` is either `true` or `false`.

`offset` refers to the maximum water depth to check for. For example, `"offset": -4` succeeds if water is 4 blocks deep or less. `offset` should almost always be 0 or a negative value. Positive values are allowed, but they still will only place blocks underneath the water, and will be functionally identical to a value of 0 (unless `surface_depth_multiplier` is set to a negative enough value to counteract it).

`surface_depth_multiplier` works exactly the same as in the `y_above` condition— it creates variation based on surface noise. Here, positive values are added to `offset`, decreasing its absolute value and thus the maximum depth, allowing the block to generate in shallower water. Negative values are subtracted from `offset`, increasing its absolute value and maximum depth, allowing the block to generate in deeper water. As with `y_above`, positive and negative integers of the same absolute value will cause equal amounts of variation, just one shifts upwards and one downwards. A value of zero leads to no variation at all.

Also like in `y_above`, `add_stone_depth`, when set to `true`, moves up to the nearest surface and checks for the maximum water depth there. You'll usually want this set to `true`; when it's set to `false` the game will still check for water at the nearest surface, but it won't take into consideration whether the block in question itself is actually submerged or not. So, when `add_stone_depth` is set to `false`, you can run into situations where you have shallow water, a few blocks of the block to place in shallow water/on dry land underneath that, and *then* blocks to be placed in deep water all the way until you reach another surface… A lot of the time, the `water` condition is used in combination with several other surface rules, such that this field doesn't actually matter (which I think is why it's so often set to `false` in vanilla world generation; or maybe setting it to `false` reduces lag), and there are some use cases where someone would actively want this set to `false`, though I myself can't think of any. If you don't know whether to set this to `true` or `false`, I would recommend setting it to `true`.

(For those interested, the exact calculation used by this condition is `y + (addStoneDepth ? stoneDepthAbove : 0) >= waterHeight + offset + surfaceDepth * surfaceDepthMultiplier`, and it succeeds when there is *not* a fluid. Thank you to jacobsjo for providing this and for explaining this entire surface rule to me.)

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": 0,
				"surface_depth_multiplier": 0,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:grass_block",
					"Properties": {
						"snowy": "false"
					}
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:dirt"
			}
		}
	]
}
```
> ^ This is a very important part of the sequence for vanilla terrain generation. What we're doing here is checking if the water is 0 blocks deep or less above the block we're looking at. Remember that the `water` condition is met if there is **not** water above the surface, so we place grass on success. Then, we place dirt elsewhere (wherever there is water directly above). We want no variation (no dirt on land and no grass under water), so `surface_depth_multiplier` is set to `0`. *(I have `add_stone_depth` set to `false` because that's how it's set in vanilla— since vanilla uses this surface rule in combination with others, `add_stone_depth` doesn't actually change anything. If you make a dimension with JUST this surface rule, changing `add_stone_depth` will have an effect, but making a dimension with just this surface rule is a bad idea anyways and could be laggy as all the grass updates.)*

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": 0,
				"surface_depth_multiplier": 0,
				"add_stone_depth": true
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:sandstone"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": -10,
				"surface_depth_multiplier": 2,
				"add_stone_depth": true
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:stone"
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:prismarine"
			}
		}
	]
}
```
> ^ Here, I want sandstone on dry land, stone in shallow water and prismarine in deep water, so I run multiple water conditions in sequence. The first checks that water is less than one block deep and places sandstone on success; I want no variation here, so I have `surface_depth_multiplier` set to `0`. The second checks that water is 10 blocks deep or less, with some variation, and places stone there. Because I do want variation here, I have `surface_depth_multiplier` set to a non-zero value. I'd rather have the maximum water depth be shallower than 10 blocks rather than deeper, so I chose a positive number for `surface_depth_multiplier` rather than a negative one. Finally, I place prismarine everywhere else (which is by this point just in deep water). I don't want to have prismarine underneath stone in shallow water, so I set `add_stone_depth` to `true`.

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:y_above",
				"anchor": {
					"above_bottom": 2
				},
				"surface_depth_multiplier": 0,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:water",
							"offset": 0,
							"surface_depth_multiplier": 0,
							"add_stone_depth": true
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:nether_brick_fence",
								"Properties": {
									"north": "true",
									"east": "true",
									"south": "true",
									"west": "true",
									"waterlogged": "false"
								}
							}
						}
					},
					{
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:nether_brick_fence",
							"Properties": {
								"north": "true",
								"east": "true",
								"south": "true",
								"west": "true",
								"waterlogged": "true"
							}
						}
					}
				]
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:bedrock",
			}
		}
	]
}
```
> ^ This sequence creates a world entirely of nether brick fences, with a couple of layers of bedrock below. I set fences underneath any water to be waterlogged, until the bottom of the world (or a cave). This of course still isn't perfect— for example waterlogged blocks also appear underneath lava— but this is technically something you could do.

  </details>

</details>

***

***

***





## **Condition — vertical_gradient**

<details>
  <summary>Limits surface rules by y-value, with a messy transition.</summary>
  <br>

Used by default for the bedrock floors, bedrock roof, and deepslate gradient, this surface rule is kind of like `y_above` but creates a messy gradient in between the two given values, isn't as configurable, and affects the bottom of the world rather than the top.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:vertical_gradient",
	"random_name": <string>,
	"true_at_and_below": {
		<vertical anchor>
	},
	"false_at_and_above": {
		<vertical anchor>
	}
}
```
where `<string>` is an arbitrary string used as the seed for the gradient,
and `<vertical anchor>` is the a vertical anchor (see `y_above` for explanation).

Note that the vertical anchor for `true_at_and_below` must be lower in the world than the vertical anchor for `false_at_and_above`, or the surface rule won't work.

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:vertical_gradient",
		"random_name": "minecraft:bedrock_floor",
		"true_at_and_below": {
			"above_bottom": 0
		},
		"false_at_and_above": {
			"above_bottom": 5
		}
	},
	"then_run": {
		"type": "minecraft:block",
		"result_state": {
			"Name": "minecraft:bedrock"
		}
	}
}
```
> ^ This is the surface rule used to create the bedrock floor in the Overworld and the Nether. We want bedrock to always exist at the bottom of the world, and no bedrock to exist more than 5 blocks above the world, so that's what we set our values to. In the space in between, there will be more bedrock lower down and less bedrock higher up. Vanilla uses the random name `"minecraft:bedrock_floor"`, but this could be pretty much anything and it will just change which blocks within the gradient are chosen to be bedrock.

> (There is be an example of the Nether's bedrock ceiling in the `not` section.)

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:bedrock_floor",
				"true_at_and_below": {
					"above_bottom": 0
				},
				"false_at_and_above": {
					"above_bottom": 5
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:bedrock"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:deepslate",
				"true_at_and_below": {
					"absolute": 0
				},
				"false_at_and_above": {
					"absolute": 8
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:deepslate",
					"Properties": {
						"axis": "y"
					}
				}
			}
		}
	]
}
```
> ^ This sequence combines the bedrock floor with the deepslate gradient from vanilla. The deepslate condition is in charge of all deepslate placement in vanilla world generation. We have bedrock come before deepslate because otherwise the deepslate would place all the way to the bottom of the world and there would be nothing left for the bedrock to replace.

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "BLAbabububwbbibibiebibbieeeiie",
				"true_at_and_below": {
					"above_bottom": 0
				},
				"false_at_and_above": {
					"above_bottom": 15
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:bedrock"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "BLAbabububwbbibibiebibbieeeiie",
				"true_at_and_below": {
					"above_bottom": 0
				},
				"false_at_and_above": {
					"above_bottom": 15
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:lime_concrete"
				}
			}
		}
	]
}
```
> ^ Here I've created a sequence that will not work as intended. Remember that `random_name` is an arbitrary string, so I can set it to `"BLAbabububwbbibibiebibbieeeiie"` if I want. However, since I set both conditions to the same anchors and the same `random_name`, they will target the same blocks to replace, meaning that no lime concrete will show up in the world. If, however, I set the second `random_name` to something like `"auuau22828wwwwwwwwwww"`, it should place a gradient of bedrock first, and then a gradient of lime concrete on the stone that survives after that.

  </details>

</details>

***

***

***





## **Condition — stone_depth**

<details>
  <summary>Limits surface rules by proximity to the surface, noise caves, etc.</summary>
  <br>

This condition is used to place blocks only on surfaces (floors and ceilings), and it's what's used to place things like grass and and sand on the world surface in vanilla, as well as the layers of dirt and sandstone underneath them. Note that using `stone_depth` alone will place blocks in noise caves **__(but not carved caves)__** as well; combine this condition with the `above_preliminary_surface` condition (see below) to get surface rules mostly limited to the uppermost blocks of terrain.

This condition ignore water and lava, so it will place blocks at the on the ocean floor *(rather than the ocean's surface or not at all)*.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:stone_depth",
	"surface_type": <"floor" or "ceiling">,
	"add_surface_depth": <true or false>,
	"secondary_depth_range": <non-negative integer>,
	"offset": <integer value>
}
```
where `<"floor" or "ceiling">` is either `"floor"` or `"ceiling"`,
`<true or false>` is either `true` or `false`,
`<non-negative integer>` is a positive integer or zero,
and `<integer value>` is an integer.

`surface_type` specifies whether you want to replace to blocks in a floor (including the world's surface) or a ceiling (including the bottom of the world, where the bedrock floor normally is!).

Normally, `stone_depth` only replaces the uppermost or lowermost block within the column, the one directly above or below air. Setting `add_surface_depth` to `true` will increase this depth by 0-4 (varies by column, but is usually 2-3), so it will replace the uppermost/lowermost 1-5 blocks.

`secondary_depth_range` is similar to `add_surface_depth`, but it uses a different noise that behaves identically (so it would add 0-4 blocks, just different values in different places), and with `secondary_depth_range`, you can choose to scale the noise up to a particular value. Specifically, the game will take the noise, and scale it to match the range from 0 to whichever integer you specify. If you want no secondary depth, you can set this value to `0`. If you want it to behave exactly like in previous versions, set this value to `6`. Vanilla only has `secondary_depth_range` set to a value other than `0` in two places: Sandstone underneath sand in the warm ocean, beach, and snowy beach biomes has a `secondary_depth_range` of `6`, and sandstone underneath sand in the desert biome has a `secondary_depth_range` of `30`. Despite its name, `secondary_depth_range` is independent of `add_surface_depth`, so you can set the `add_surface_depth` to `false` and still set a `secondary_depth_range`. `add_surface_depth` and `secondary_depth_range` are additive, so using both will cause the depth to be deeper than either alone.

`offset` adds or subtracts the specified integer value to or from the depth with no variation. So, for example, if you want your dirt to range from 3-7 blocks deep instead of 1-5, you could set this value to `2`. `offset` can also be negative, which will decrease the depth by a constant value. If the total depth in a column goes to zero or is negative, no blocks are placed. *(Remember that this condition by default starts with a depth of 1, so when `add_surface_depth` is set to `false` and `add_surface_secondary_depth` is set to `0`— and there are no other blocks like grass to take into account— an `offset` of `0` keeps the usual distribution, but an `offset` of `-1` or more will create a greater proportion of columns with no blocks.)* `offset` is independent of and additive with `add_surface_depth` and `secondary_depth_range`. Positive `offset` values will still increase the number of blocks in when `surface_type` is set to `ceiling` (and negative values will still decrease the number of blocks).

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "floor",
				"add_surface_depth": false,
				"secondary_depth_range": 0,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:water",
							"offset": 0,
							"surface_depth_multiplier": 0,
							"add_stone_depth": "false"
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:grass_block",
								"Properties": {
									"snowy": "false"
								}
							}
						}
					},
					{
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:dirt"
						}
					}
				]
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "floor",
				"add_surface_depth": true,
				"secondary_depth_range": 0,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:dirt"
				}
			}
		}
	]
}
```
> ^ This surface rule combines the sequence that places grass above water and dirt below water with the `stone_depth` condition to create standard grass and dirt layers. Specifically, this sequence checks the topmost block of every floor (the world's surface and the floors of noise caves), and if that block is not under water or lava, it places grass; if it is under water or lava, it places dirt. Then, this sequence places dirt in the top few blocks of every floor (since we already placed grass or dirt and neither of those are the default block, this does not overwrite those). The rest of the world is our default block (stone).

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "floor",
				"add_surface_depth": true,
				"secondary_depth_range": 0,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:sand"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "floor",
				"add_surface_depth": true,
				"secondary_depth_range": 6,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:sandstone"
				}
			}
		}
	]
}
```
> ^ This is a similar sequence. Here, however, we place a few blocks of sand, and then another few blocks of sandstone underneath that. The rest of the world is still stone.

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:lush_caves"
		]
	},
	"then_run": {
		"type": "minecraft:sequence",
		"sequence": [
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "ceiling",
					"add_surface_depth": false,
					"secondary_depth_range": 0,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:moss_block"
					}
				}
			},
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": false,
					"secondary_depth_range": 3,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:moss_block"
					}
				}
			}
		]
	}
}
```
> ^ Here, we place moss blocks on the ceilings and floors of lush caves. I want the moss in the floor to be only a couple of blocks deep, but I still want a little variation, so I use `secondary_depth_range` with a low value instead of `add_surface_depth.

  </details>

  <details>
    <summary>Example 4</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:biome",
		"biome_is": [
			"minecraft:basalt_deltas"
		]
	},
	"then_run": {
		"type": "minecraft:sequence",
		"sequence": [
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": true,
					"secondary_depth_range": 10,
					"offset": 5
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:blackstone"
					}
				}
			},
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "ceiling",
					"add_surface_depth": true,
					"secondary_depth_range": 10,
					"offset": 5
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:blackstone"
					}
				}
			},
		]
	}
}
```
> ^ Here, I want there to be blackstone in the floor and the ceiling, with a lot of depth variation, but always having at least several blocks. To ensure that I always have at least 6 blocks of blackstone, I set the `offset` to `5` (remember that this condition starts with a depth of 1). I also used both `add_surface_depth` and a high `secondary_depth_range`, so that the variation would be more noticeable on such a large scale.

  </details>

</details>

***

***

***





## **Condition — hole**

<details>
  <summary>Limits surface rules to columns with 0 surface noise.</summary>
  <br>

This condition affects columns where the surface noise (the one used for `add_surface_depth` in the `stone_depth` condition) is equal to 0; i.e. where there would be no dirt underneath grass in vanilla. Keep in mind that *this affects all y-levels*, so you'll usually want to limit this with other conditions. (Also keep in mind this this condition has no relation to the noise used for `secondary_depth_range` in the `stone_depth` condition, so you could still get blocks from that if you set it to a positive value.)

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:hole"
}
```
(This condition only has the `"type"` field.)

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:hole"
	},
	"then_run": {
		"type": "minecraft:block",
		"result_state": {
			"Name": "minecraft:air"
		}
	}
}
```
> ^ This simple surface rule places air wherever the surface noise is 0, creating pits that fall straight into the void.

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:y_above",
		"anchor": {
			"absolute": 63,
		},
		"add_stone_depth": false,
		"surface_depth_multiplier": 0
	},
	"then_run": {
		"type": "minecraft:sequence",
		"sequence": [
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:hole"
				},
				"then_run": {
					"type": "minecraft:condition",
					"if_true": {
						"type": "minecraft:stone_depth",
						"surface_type": "floor",
						"add_surface_depth": false,
						"secondary_depth_range": 0,
						"offset": 0
					},
					"then_run": {
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:air"
						}
					}
				}
			},
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:water",
					"offset": 0,
					"surface_depth_multiplier": 0,
					"add_stone_depth": false
				},
				"then_run": {
					"type": "minecraft:condition",
					"if_true": {
						"type": "minecraft:stone_depth",
						"surface_type": "floor",
						"add_surface_depth": false,
						"secondary_depth_range": 0,
						"offset": 0
					},
					"then_run": {
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:grass_block",
							"Properties": {
								"snowy": "false"
							}
						}
					}
				}
			},
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": true,
					"secondary_depth_range": 0,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:dirt"
					}
				}
			}
		]
	}
}
```
> ^ This is the best application of this condition I've seen, courtesy of not_crim_safe. Essentially, it looks for "holes" above sea level, and instead of placing grass, it places air. This creates patches of exposed stone with no dirt in between, while still allowing dirt to generate underneath grass normally. We don't need to worry about the dirt placing underneath the air because the air will only place by definition in locations where there is no surface depth anyways.

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "ceiling",
				"add_surface_depth": false,
				"secondary_depth_range": 0,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:netherrack"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:hole"
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": false,
					"secondary_depth_range": 6,
					"offset": -2
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:lava",
						"Properties": {
							"level": "0"
						}
					}
				}
			}
		},
	  	{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:hole"
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": false,
					"secondary_depth_range": 0,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:lava",
						"Properties": {
							"level": "0"
						}
					}
				}
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:netherrack"
			}
		}
	]
}
```
> ^ With this surface rule I have in mind lava pools for a Nether-like landscape. I don't want the lava to flow down in giant lavafalls in the middle of solid terrain though, so I start by placing netherrack one block thick on every ceiling. Next I have the lava pools, which I would like to be shallow but not uniformly 1 block deep. I can't use `add_surface_depth` because the `hole` condition by definition only places where that field will have no effect, but we *can* use `secondary_depth_range`, because that has a different noise map and has no relation to the `hole` condition. As I said though, I still want it to be shallow, so I set `offset` to -2. That would leave us with exposed netherrack in some places though, so I run another surface rule placing 1 block of lava everywhere there is a hole, to be safe. Finally, I make every other block netherrack.

  </details>

  <details>
    <summary>Example 4</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "ceiling",
				"add_surface_depth": false,
				"secondary_depth_range": 0,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:stone"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:hole"
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:y_above",
					"anchor": {
						"absolute": 73
					},
					"surface_depth_multiplier": 0,
					"add_stone_depth": true
				},
				"then_run": {
					"type": "minecraft:sequence",
					"sequence": [
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:stone_depth",
								"surface_type": "floor",
								"add_surface_depth": false,
								"secondary_depth_range": 4,
								"offset": 5
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:air"
								}
							}
						},
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:stone_depth",
								"surface_type": "floor",
								"add_surface_depth": false,
								"secondary_depth_range": 4,
								"offset": 6
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:pointed_dripstone",
									"Properties": {
										"thickness": "tip",
										"vertical_direction": "up",
										"waterlogged": "false"
									}
								}
							}
						},
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:stone_depth",
								"surface_type": "floor",
								"add_surface_depth": false,
								"secondary_depth_range": 4,
								"offset": 7
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:pointed_dripstone",
									"Properties": {
										"thickness": "tip_merge",
										"vertical_direction": "up",
										"waterlogged": "false"
									}
								}
							}
						},
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:stone_depth",
								"surface_type": "floor",
								"add_surface_depth": false,
								"secondary_depth_range": 4,
								"offset": 8
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:pointed_dripstone",
									"Properties": {
										"thickness": "frustum",
										"vertical_direction": "up",
										"waterlogged": "false"
									}
								}
							}
						},
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:stone_depth",
								"surface_type": "floor",
								"add_surface_depth": false,
								"secondary_depth_range": 4,
								"offset": 9
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:pointed_dripstone",
									"Properties": {
										"thickness": "base",
										"vertical_direction": "up",
										"waterlogged": "false"
									}
								}
							}
						}
					]
				}
			}
		}
	]
}
```
> ^ This delightfully devilish surface rule creates pits with pointed dripstone where "holes" are. I again use a ceiling placement to prevent the pointed dripstone from floating. I also limit the pointed dripstone pits to a sufficiently high y-value (so that we don't run into weird stuff with the ocean, rivers, and aquifers).

  </details>

</details>

***

***

***





## **Condition — above_preliminary_surface**

<details>
  <summary>Limits surface rules to the surface of the world.</summary>
  <br>

This condition allows blocks to be replaced near-ish to the surface. It's useful for when you want blocks to place on the earth's surface but not in most caves. These blocks will stay take appear in some noise caves within several blocks of the world's actual surface (in my brief testing, this averages ~20), but most of the caves should be unaffected.

*(For those interested in modifying this, who have some familiarity with the terrain shaper, jacobsjo has estimated the value of `preliminary_surface_level` to be roughly calculated by the expression `(offset + 0.5 - (0.2734375 / factor)) * 128)` (note that this formula is for an earlier pre-release; it might have changed by now as there has been at least one change to `preliminary_surface_level` since then). He also discovered that the `preliminary_surface_level` changes at intervals of `size_vertical * 4`— although it is smoothed— which means that with default settings you will get plateaus around every 8 blocks with slopes in between. He also discovered that the `above_preliminary_surface` surface rule takes effect on any blocks that are 8 blocks below the `preliminary_surface_level` or above, but because `preliminary_surface_level` is not parallel with the actual height of the world's surface, the real variation is quite great. Regardless, this surface rule is the best way we currently have to say "only do this at the surface of the world". (We can't use `noise_threshold` with depth because depth— although a biome placement parameter— is not a noise.))*

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:above_preliminary_surface"
}
```
(This condition only has the `"type"` field.)

  <h3> Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:bedrock_floor",
				"true_at_and_below": {
					"above_bottom": 0
				},
				"false_at_and_above": {
					"above_bottom": 5
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:bedrock"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:above_preliminary_surface"
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": false,
					"secondary_depth_range": 0,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:sequence",
					"sequence": [
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:water",
								"offset": 0,
								"surface_depth_multiplier": 0,
								"add_stone_depth": "false"
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:grass_block",
									"Properties": {
										"snowy": "false"
									}
								}
							}
						},
						{
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:dirt"
							}
						}
					]
				}
			},
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": true,
					"secondary_depth_range": 0,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:dirt"
					}
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:deepslate",
				"true_at_and_below": {
					"absolute": 0
				},
				"false_at_and_above": {
					"absolute": 8
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:deepslate",
					"Properties": {
						"axis": "y"
					}
				}
			}
		}
	]
}
```
> ^ Combining a bit of many things we've learned, this is a basic surface rule for a generic overworld with one biome. I start by placing the bedrock floor because it's by the most important surface rule (I don't want any other surface rules to make holes in the bedrock floor). Next, I have the grass and dirt surface rule, because on the astronomically low chance that the world's surface generates down to deepslate level, I would rather have a grass floor than a deepslate one. This grass and dirt surface rule is limited to be above the preliminary surface, so most caves should be free of grass and dirt (except that coming from blobs). Finally, I have the deepslate surface rule.

  </details>

</details>

***

***

***





## **Condition — noise_threshold**

<details>
  <summary>Limits surface rules to specified perlin noise ranges.</summary>
  <br>

This condition places blocks if the block at this position falls within a range of values on a noise map generated for the world. This can be a noise already in the game or one you make yourself (so long as it exists as a file in your datapack)— and those already in the game can be arbitrary (like `calcite`) or used in other parts of world generation (like `ridge`), the latter of which lets you match up surface rules to other terrain patterns. You can get quite different results depending on what values you specify and the `firstOctave` and `amplitudes` within the noise file.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:noise_threshold",
	"noise": <noise>,
	"min_threshold": <decimal value>,
	"max_threshold": <decimal value>
}
```
where `<noise>` is the namespaced ID of a noise file (not a noise settings file!),
and `<decimal value>` is a positive or negative number (or zero) with a decimal point.

(The thresholds are inclusive, so if you set them equal to 0.0 and 0.5, for example, your surface rule will take effect at both 0.0 and 0.5.

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:above_preliminary_surface"
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:stone_depth",
			"surface_type": "floor",
			"add_surface_depth": false,
			"secondary_depth_range": 0,
			"offset": 0
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": 0,
				"surface_depth_multiplier": 0,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:biome",
					"biome_is": [
						"minecraft:stony_peaks"
					]
				},
				"then_run": {
					"type": "minecraft:condition",
					"if_true": {
						"type": "minecraft:noise_threshold",
						"noise": "minecraft:calcite",
						"min_threshold": -0.0125,
						"max_threshold": 0.0125
					},
					"then_run": {
						"type": "minecraft:block"
						"result_state": {
							"Name": "minecraft:calcite"
						}
					}
				}
			}
		}
	}
}
```
> ^ This is the surface rule in charge of placing calcite strips in the stony peaks biome. After checking that we're at/near the surface of the world, it checks the uppermost block of floors that are *not* submerged in water, within the stony peaks biome, to see if the block's multinoise value for the `"minecraft:calcite"` noise is within the given range: if it is, the block is calcite. Because the range given straddles `0.0`, we end up with long, winding strips of the material (think spaghetti caves).

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:above_preliminary_surface"
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:stone_depth",
			"surface_type": "floor",
			"add_surface_depth": false,
			"secondary_depth_range": 0,
			"offset": 0
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:biome",
				"biome_is": [
					"furbyoverworldredux:tropical_upland_river",
					"furbyoverworldredux:temperate_upland_river",
					"furbyoverworldredux:subpolar_taiga_river",
					"furbyoverworldredux:polar_taiga_river"
				]
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:noise_threshold",
							"noise": "furbyoverworldredux:river_erosion",
							"min_threshold": 0.0,
							"max_threshold": 1E308
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:coarse_dirt"
							}
						}
					},
					{
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:cobblestone"
						}
					}
				]
			}
		}
	}
}
```
> ^ This is (part of) my surface rule I use in my custom biomes. As you can see, you can use both custom biomes and custom noise maps. Here, I place coarse dirt where the threshold matches, and cobblestone everywhere else. However, I define this range as zero to infinity. *(Note that `1E308` is a comically large overkill value, and so for our purposes might as well be infinity. Most noises will barely pass -1.0 and 1.0, and even my most extreme custom one doesn't seem to pass -5.0 and 5.0. Still, it's easy (and funny) to not worry about that and just set the max threshold to infinity where desired.))* Because I set a range to be "greater than zero" rather than "around zero", we will end up not with strips but with blobs (think cheese caves, although the noise I'm using is on a much smaller scale).

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:bedrock_floor",
				"true_at_and_below": {
					"above_bottom": 0
				},
				"false_at_and_above": {
					"above_bottom": 5
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:bedrock"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:deepslate_floor",
				"true_at_and_below": {
					"above_bottom": 8
				},
				"false_at_and_above": {
					"above_bottom": 11
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:deepslate",
					"Properties": {
						"axis": "y"
					}
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "floor"
				"add_surface_depth": true,
				"secondary_depth_range": 6,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:stone"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"surface_type": "ceiling"
				"add_surface_depth": true,
				"secondary_depth_range": 6,
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:stone"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:noise_threshold",
				"noise": "minecraft:cave_cheese",
				"min_threshold": 0.45,
				"max_threshold": 1E308
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:stone_depth",
							"surface_type": "floor"
							"add_surface_depth": true,
							"secondary_depth_range": 6,
							"offset": 0
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:stone"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:stone_depth",
							"surface_type": "ceiling"
							"add_surface_depth": true,
							"secondary_depth_range": 6,
							"offset": 0
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:stone"
							}
						}
					}
				]
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:noise_threshold",
				"noise": "minecraft:cave_cheese",
				"min_threshold": 0.45,
				"max_threshold": 0.5
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:wet_sponge"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:noise_threshold",
				"noise": "minecraft:cave_cheese",
				"min_threshold": 0.5,
				"max_threshold": 1E308
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:air"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:deepslate_floor",
				"true_at_and_below": {
					"absolute": 0
				},
				"false_at_and_above": {
					"absolute": 8
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:deepslate",
					"Properties": {
						"axis": "y"
					}
				}
			}
		}
	]
}
```
> ^ You can also have noises referenced which are used in other parts of terrain generation. Here, I want there to be air far away from cheese caves to create even more "caves". So I define a noise threshold that's limited to be far away from cheese caves, though it can still be intersected by spaghetti caves, noodle caves, ore veins, cave carvers, structures, etc. I have these "caves" marked by sponge borders. *(I will be the first to admit that this example is very poorly done; the "caves" are icky nearly-uniform columns from just under the surface of the world down tonearly  bedrock; this is more of a proof of concept. As said in the introduction, Ganar has made custom noise caves that look gorgeous, so actually good caves are still in the realm of possibility. And now, with density functions, there are much, much more efficient ways to make your own caves.)*

  </details>

</details>

***

***

***





## **Condition — temperature**

<details>
  <summary>Limits surface rules to some parts of the frozen ocean and deep frozen ocean biomes, and to the entirety of other frozen biomes.</summary>
  <br>

To my understanding, this condition checks the block's temperature value (the actual temperature value determining rain vs. snowfall, not the noise temperature value used in biome placement), and if it is at a value such that it snows regardless of y-level (specifically, it the temperature is less than 0.15), the condition succeeds.

In vanilla, this only shows up in one place, but as far as I know (~80% certainty) it is never actually called (it's part of a sequence where the surface rule before it has a condition technically different but I think functionally identical to a condition on the sequence as a whole).
The intent had something to do with placing blocks in certain parts of frozen ocean and deep frozen ocean biomes (this would work because those biomes are hard-coded to have variable temperature)— this functionality is still possible, although the variable temperature characteristic of these biomes cannot be given to other biomes (including custom biomes), so you're going to have to use one or both of these biomes if you want that kind of functionality.

Because this condition has no configurable fields, the only application outside of frozen oceans and deep frozen oceans is to use this to run a surface rule in every frozen biome without having to actually specify all of those biomes.

(You **_cannot_** use this condition to limit surface rules to be above the snow-line in biomes where it rains or snows depending on height— I tried.)

  <h3>Format:</h3>

```json
{
	"type": "minecraft:temperature"
}
```
(This condition only has the `"type"` field.)

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:above_preliminary_surface"
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:temperature"
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": 0,
				"surface_depth_multiplier": 0,
				"add_stone_depth": true
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"surface_type": "floor",
					"add_surface_depth": false,
					"secondary_depth_range": 0,
					"offset": 0
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:snow_block"
					}
				}
			}
		}
	}
}
```
> ^ Here's the simpler possible application of this condition: running surface rules in every constantly snowy biome without specifying all of them. For this particular example, I'm replacing every dry uppermost block on the surface the world with snow, so long as the biome is a snowy one (or so long as we are in the snowy parts of frozen oceans and deep frozen oceans).

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:y_above",
		"anchor": {
			"absolute": 62
		},
		"surface_depth_multiplier": -2,
		"add_stone_depth": false
	},
	"then_run": {
		"type": "minecraft:sequence",
		"sequence": [
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:temperature"
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:packed_ice"
					}
				}
			},
			{
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:y_above",
					"anchor": {
						"absolute": 63
					},
					"surface_depth_multiplier": 0,
					"add_stone_depth": false
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:air"
					}
				}
			},
			{
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:water",
					"Properties": {
						"level": "0"
					}
				}
			}
		]
	}
}
```
> ^ This sequence removes islands in frozen ocean and deep frozen ocean biomes, replacing them either with air and water (where it rains) or packed ice (where it snows).

  </details>

</details>

***

***

***





## **Condition — steep**

<details>
  <summary>Limits surface rules to steep faces on the north and east sides of mountains.</summary>
  <br>

The `steep` condition succeeds at any position where there's a vertical gradient greater than 2 on its north and/or east face. Also, at a chunk boundary, the gradient needs to be greater than 4. (Thank you jacobsjo for this explanation.)

The steepness is calculated at the world's surface (**NOT** a cave surface— this always uses the uppermost block and excludes noise caves), and unless limited by other conditions, will carry on throughout the entire column. This condition used by vanilla in frozen peaks, jagged peaks, and snowy slopes, to place packed ice (frozen peaks) and stone (jagged peaks and snowy slopes).

For blocks not on a chunk border, the exact calculation is as follows (thank you Apollo for figuring this out): The heightmap gets sampled 1 block south and 1 block north of the block being checked by the surface rule. If the block to the south is 4 or more blocks higher than the block to the north, the condition succeeds. If this check does not pass, the same process happens with the blocks immediately east and west— with west needing to be 4 or more blocks higher than east— and if this check passes, then the condition still succeeds. If both checks do not pass, the condition fails. (At chunk borders, the game will instead sample the original block itself in the place of any blocks that would be outside the chunk.)

**Remember that this condition only works on two sides of mountains, not all four of them.**

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:steep"
}
```
(This condition only has the `"type"` field.)

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:above_preliminary_surface"
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:stone_depth",
			"surface_type": "floor",
			"add_surface_depth": false,
			"secondary_depth_range": 0,
			"offset": 0
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": 0,
				"surface_depth_multiplier": 0,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:biome",
					"biome_is": [
						"minecraft:jagged_peaks"
					]
				},
				"then_run": {
					"type": "minecraft:sequence",
					"sequence": [
						{
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:steep"
							},
							"then_run": {
								"type": "minecraft:block",
								"result_state": {
									"Name": "minecraft:stone"
								}
							}
						},
						{
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:snow_block"
							}
						}
					]
				}
			}
		}
	}
}
```
> ^ Here's the surface rule for stone faces in the jagged peaks biome. For each dry uppermost block at the surface of the world in the jagged peaks biome, the game places stone where it's sufficiently steep on the north and east faces of the mountain, and snow blocks everywhere else (where it's steep on the south and west faces, and where it isn't steep at all).

  </details>

</details>

***

***

***





## **Condition — not**

<details>
  <summary>Limits surface rules to the inverse of another condition.</summary>
  <br>

`not` is one of the simplest but one of the most powerful conditions. It references any other condition, and succeeds wherever that condition would fail. If when learning about one of the above conditions you thought, "huh, I wish I could do the opposite of that", add a `not` before it and you can.

  <h3>Formatting:</h3>

```json
{
	"type": "minecraft:not",
	"invert": {
		<condition>
	}
}
```
where `<condition>` is another surface rule condition (including all of its fields).

  <h3>Examples:</h3>

  <details>
    <summary>Example 1</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:not",
		"invert": {
			"type": "minecraft:vertical_gradient",
			"random_name": "minecraft:bedrock_roof",
			"true_at_and_below": {
				"below_top": 5
			},
			"false_at_and_above": {
				"below_top": 0
			}
		}
	},
	"then_run": {
		"type": "minecraft:block",
		"result_state": {
			"Name": "minecraft:bedrock"
		}
	}
}
```
> ^ This is the surface rule in charge of placing the bedrock ceiling in the Nether. By inverting it, `true_at_and_below` really means "false at and below", and `false_at_and_above` really means "true at and above". The gradient is effectively flipped (more bedrock the higher up you go).

  </details>

  <details>
    <summary>Example 2</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:above_preliminary_surface"
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:stone_depth",
			"surface_type": "floor",
			"add_surface_depth": false,
			"secondary_depth_range": 0,
			"offset": 0
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:water",
				"offset": 0,
				"surface_depth_multiplier": 0,
				"add_stone_depth": false
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:biome",
					"biome_is": [
						"minecraft:stony_peaks"
					]
				},
				"then_run": {
					"type": "minecraft:condition",
					"if_true": {
						"type": "minecraft:not",
						"invert": {
							"type": "minecraft:steep"
						}
					},
					"then_run": {
						"type": "minecraft:block",
						"result_state": {
							"Name": "minecraft:moss_block"
						}
					}
				}
			}
		}
	}
}
```
> ^ This is a surface rule placing moss blocks where on the steep south and west faces of stony peaks biomes (rather than north and east) *and* wherever it's not steep at all. By doing this, you don't have to manually place stone on the north and east faces, preserving them to potentially be changed by later surface rules.

> (Sadly, there is no way to place blocks on *just* the steep south and east faces, or *just* where it's not steep, because the `steep` condition checks for both within a single condition.)

  </details>

  <details>
    <summary>Example 3</summary>

```json
{
	"type": "minecraft:sequence",
	"sequence": [
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:vertical_gradient",
				"random_name": "minecraft:bedrock_floor",
				"true_at_and_below": {
					"above_bottom": 0
				},
				"false_at_and_above": {
					"above_bottom": 5
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:bedrock"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:invert",
				"not": {
					"type": "minecraft:vertical_gradient",
					"random_name": "minecraft:bedrock_roof",
					"true_at_and_below": {
						"below_top": 5
					},
					"false_at_and_above": {
						"below_top": 0
					}
				}
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:bedrock"
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:not",
				"invert": {
					"type": "minecraft:water",
					"offset": 0,
					"surface_depth_multiplier": 0,
					"add_stone_depth": true
				}
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:stone_depth",
					"add_surface_depth": true,
					"secondary_depth_range": 0,
					"surface_type": "floor",
					"offset": 1
				},
				"then_run": {
					"type": "minecraft:block",
					"result_state": {
						"Name": "minecraft:magma_block"
					}
				}
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"add_surface_depth": true,
				"secondary_depth_range": 0,
				"surface_type": "floor",
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:biome",
							"biome_is": [
								"minecraft:basalt_deltas"
							]
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:basalt",
								"Properties": {
									"axis": "y"
								}
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:biome",
							"biome_is": [
								"minecraft:soul_sand_valley"
							]
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:soul_sand"
							}
						}
					}
				]
			}
		},
		{
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:stone_depth",
				"add_surface_depth": false,
				"secondary_depth_range": 0,
				"surface_type": "floor",
				"offset": 0
			},
			"then_run": {
				"type": "minecraft:sequence",
				"sequence": [
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:biome",
							"biome_is": [
								"minecraft:crimson_forest"
							]
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:crimson_nylium"
							}
						}
					},
					{
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:biome",
							"biome_is": [
								"minecraft:warped_forest"
							]
						},
						"then_run": {
							"type": "minecraft:block",
							"result_state": {
								"Name": "minecraft:warped_nylium"
							}
						}
					}
				]
			}
		},
		{
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:netherrack"
			}
		}
	]
}
```
> ^ Here I have in mind a surface rule for a simpler Nether, but I want there to be a few blocks of magma underneath the lava ocean. I can invert the `water` condition to place magma underneath the lava without having to specify every dry block first.

  </details>

  <details>
    <summary>Example 4</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:not",
		"invert": {
			"minecraft:biome",
			"biome_is": [
				"minecraft:stony_shore",
				"minecraft:stony_peaks",
				"minecraft:snowy_slopes",
				"minecraft:frozen_peaks",
				"minecraft:badlands",
				"minecraft:wooded_badlands",
				"minecraft:eroded_badlands"
			]
		}
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:stone_depth",
			"surface_type": "floor",
			"add_surface_depth": false,
			"secondary_depth_range": 0,
			"offset": 0
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:noise_threshold",
				"noise": "arealdatapack:sand",
				"min_threshold": 0.8,
				"max_threshold": 1E308
			},
			"then_run": {
				"type": "minecraft:block",
				"result_state": {
					"Name": "minecraft:sand"
				}
			}
		}
	}
}
```
> ^ Here, I want little surface blobs of sand in most but not all biomes. It's much quicker for me to define the biomes I *don't* want this surface rule to appear in than those that I do.

  </details>

  <details>
    <summary>Example 5</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:not",
		"invert": {
			"type": "minecraft:stone_depth",
			"surface_type": "floor",
			"add_surface_depth": true,
			"secondary_depth_range": 6,
			"offset": 0
		}
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:not",
			"invert": {
				"type": "minecraft:stone_depth",
				"surface_type": "ceiling",
				"add_surface_depth": true,
				"secondary_depth_range": 6,
				"offset": 0
			}
		},
		"then_run": {
			"type": "minecraft:block",
			"result_state": {
				"Name": "minecraft:air"
			}
		}
	}
}
```
> ^ If you want to run surface rules on inverse conditions (e.g. places that *aren't* floors and ceilings), without jeopardizing your ability to run surface rules on the conditions themselves (e.g. floors and ceilings) in the future, you can use the `not` condition.

  </details>

  <details>
    <summary>Example 6</summary>

```json
{
	"type": "minecraft:condition",
	"if_true": {
		"type": "minecraft:stone_depth",
		"surface_type": "floor",
		"add_surface_depth": false,
		"secondary_depth_range": 0,
		"offset": 0
	},
	"then_run": {
		"type": "minecraft:condition",
		"if_true": {
			"type": "minecraft:noise_threshold",
			"noise": "minecraft:erosion",
			"min_threshold": -0.78,
			"max_threshold": 0.55
		},
		"then_run": {
			"type": "minecraft:condition",
			"if_true": {
				"type": "minecraft:not",
				"invert": {
					"type": "minecraft:noise_threshold",
					"noise": "minecraft:ridge",
					"min_threshold": -0.77
					"max_threshold": -0.56
				}
			},
			"then_run": {
				"type": "minecraft:condition",
				"if_true": {
					"type": "minecraft:not",
					"invert": {
						"type": "minecraft:noise_threshold",
						"noise": "minecraft:ridge"
						"min_threshold": 0.56,
						"max_threshold": 0.77
					}
				},
				"then_run": {
					"type": "minecraft:condition",
					"if_true": {
						"type": "minecraft:not",
						"invert": {
							"type": "minecraft:noise_threshold",
							"noise": "minecraft:ridge",
							"min_threshold": -0.06,
							"max_threshold": 0.06
						}
					},
					"then_run": {
						"type": "minecraft:condition",
						"if_true": {
							"type": "minecraft:not",
							"invert": {
								"type": "minecraft:noise_threshold",
								"noise": "minecraft:continentalness",
								"min_threshold": -1.06,
								"max_threshold": -0.1
							}
						},
						"then_run": {
							"type": "minecraft:condition",
							"if_true": {
								"type": "minecraft:noise_threshold",
								"noise": "minecraft:vegetation",
								"min_threshold": -0.25,
								"max_threshold": 2.0
							},
							"then_run": {
								"type": "minecraft:condition",
								"if_true": {
									"type": "minecraft:noise_threshold",
									"noise": "arealdatapack:water_source",
									"min_threshold": 0.9,
									"max_threshold": 1E308
								},
								"then_run": {
									"type": "minecraft:block",
									"result_state": {
										"Name": "minecraft:water",
										"Properties": {
											"level": 0
										}
									}
								}
							}
						}
					}
				}
			}
		}
	}
}
```
> ^ Here, I want a surface rule to run that places water source blocks, but only when the location matches certain worldgen parameters (in addition to its own noise threshold), irrespective of biome. It's way more efficient for many of these noise thresholds to define where the water *shouldn't* be, rather than every combination of parameters that is allowed.

  </details>

</details>

***

***

***
