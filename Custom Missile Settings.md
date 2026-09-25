# Custom Missile Settings

To add a menu to the base game, use `IKUtils.AddCustomMissileMenu(CustomMissileMenuConfig config)` in your mod's entry point `preload`.

## `CustomMissileMenuConfig`

| Field | Description |
|---|---|
| `settingsType` | The type of your `SeekerSettings` class, e.g. `typeof(CruiseCmdSeekerDescriptor.CruiseCmdSeekerSettings)`. **Note:** this type is automatically added to the game's serializer. |
| `settingsPaneConfig` | The configuration for your menu. Takes a `MissileSettingsPaneConfig`. |

## `MissileSettingsPaneConfig`

| Field | Description |
|---|---|
| `string _name` | The name of the menu, e.g. `"Seeker - DataLink"`. Must match the Settings Panel Name in your Seeker Descriptor. |
| `Type _settingsType` | The script that runs on the settings menu. Must derive from `MissileBaseSeekerSettings<YourDescriptorClass>`. See the base game's `FleetEditor.MissileEditor.MissileAntiRadiationSeekerSettings` for an example. |
| `IEnumerable<ButtonConfig> _buttons` | An array of buttons your new menu will have. |

## `ButtonConfig`

| Field | Description |
|---|---|
| `string _lableText` | The label of the button, e.g. `"Only Track Prioed Targets"`. |
| `string _tooltip` | The tooltip shown when hovering over the button in the editor. Usually explains what the options do. |
| `string _buttonFieldName` | Must match the exact name of the field storing your button in the `SeekerSettings` script, e.g. `"_onlyTrackPrioed"`. |
| `string _setFunctionName` | Must match the exact name of the function called when the button is pressed, on the `SeekerSettings` script. |
| `IEnumerable<SequentialButton.SequenceOption> _options` | Base game button options — just a text label and a color. |

## Example

The `SeekerSettings` script — the menu config is stored on the script itself as a static field:

```csharp
public class MissileCruiseCmdSeekerSettings : MissileBaseSeekerSettings<CruiseCmdSeekerDescriptor>
{
    [SerializeField]
    private SequentialButton _onlyTrackPrioed;

    public void ButtonSetOnlyTrackPrioed(int prioed) => this._component.OnlyTrackPrioed = prioed == 1;

    protected override void HandleSeekerChanged(MissileComponentDescriptor component)
    {
        base.HandleSeekerChanged(component);
        this._onlyTrackPrioed.SetOptionWithoutNotify(this._component.OnlyTrackPrioed ? 1 : 0);
    }

    // Declared BEFORE DataLinkMenuConfig — static field initializers run
    // top to bottom, and DataLinkMenuConfig references this field.
    private static readonly MissileSettingsPaneConfig.ButtonConfig OnlyTargetPrioedBtnCfg = new MissileSettingsPaneConfig.ButtonConfig(
        "Only Track Prioed Targets",
        "Controls whether or not the seeker will pickup unpriorized targets. \n\nAny Tracks: Seeker will pickup and target the first track it can see, if there are multipe in its cone of vison when it activates it will pick the closest prioed track if none are found it will pick the first normal track. \n\nOnly Prioed Tracks: The missile will only accept Prioed targets, when its seeker activates it will target the closest prioed track",
        "_onlyTrackPrioed",
        "ButtonSetOnlyTrackPrioed",
        new SequenceOption[] {
            new SequenceOption() { Text = "Any Tracks", TextColor = Utility.GameColors.ColorName.Green },
            new SequenceOption() { Text = "Only Prioed Tracks", TextColor = Utility.GameColors.ColorName.Yellow }
        });

    static public readonly IKUtils.CustomMissileMenuConfig DataLinkMenuConfig = new IKUtils.CustomMissileMenuConfig()
    {
        settingsType = typeof(CruiseCmdSeekerDescriptor.CruiseCmdSeekerSettings),
        settingsPaneConfig = new MissileSettingsPaneConfig(
            "Seeker - DataLink",
            typeof(MissileCruiseCmdSeekerSettings),
            new MissileSettingsPaneConfig.ButtonConfig[] { OnlyTargetPrioedBtnCfg }
        )
    };
}
```

Inside your mod's entry point, register the menu:

```csharp
IKUtils.AddCustomMissileMenu(MissileCruiseCmdSeekerSettings.DataLinkMenuConfig);
```
