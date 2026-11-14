# Nuclear Option Bindings

Key and button bindings for Nuclear Option BepInEx 5 mods that work with keyboards, mice, gamepads and HOTAS. Players set them in ConfigurationManager (F1) by pressing the control. This is an unofficial community library.

BepInEx's `KeyboardShortcut` reads Unity's legacy input. Nuclear Option reads controllers through Rewired, and Unity's `JoystickButtonN` codes do not see them, so a gamepad button bound to a `KeyboardShortcut` does nothing. This library reads keys and mouse buttons through Unity input, the same as BepInEx, and every controller through the game's Rewired.

It is source only. The code compiles into your mod, so players install no extra DLL and your mod gets no plugin dependency. Two mods that use different versions do not conflict.

## What players see in F1

Each action is one setting. It lists the bound controls by name, each with **Remove**, and an **Add binding** button. The manager's reset button restores your default.

**Add binding** listens to every device at once for 15 seconds. Escape, closing F1 or losing focus cancels it.

| The player presses | Stored as | Shown as |
| --- | --- | --- |
| A key, with Ctrl, Shift or Alt held for a combination | `Ctrl+F7` | Ctrl + F7 |
| A modifier on its own (released without another key) | `RightControl` | RightControl |
| Mouse button 3 to 7 (left and right click operate F1) | `Mouse3` | Mouse button 4 |
| A button, bumper, trigger, D-pad direction, stick click, Start or Back on a gamepad that Rewired recognizes | `Gamepad:ButtonSouth` | Gamepad A / Cross |
| A button, POV hat direction or axis direction past halfway on any other device (HOTAS, wheels, button boxes) | `Device:Joystick\|guid\|...` | VKBsim Gladiator EVO R: Button 24 |

Capture waits until the device's axes are still for 0.3 seconds and takes only a fresh press: a control that is already held must be released first. A control that another binding of the same mod already uses is rejected.

A setting holds its rows separated by `;`, so players can also edit it in the `.cfg` file. Rows that cannot be read stay visible as "Unreadable" and are ignored; the other rows still work.

## Behavior

- **Additive.** A binding never takes the control away from the game's own bindings.
- **Ignored** without game focus, while a text field or chat has focus, while F1 or the pause menu is open, while loading, and while the full map is open (unless the binding allows it).
- **Exact modifiers.** `Ctrl+F7` needs Ctrl (either side) and no Shift or Alt. `F7` does not fire while Ctrl, Shift or Alt is held. Other held keys do not block a binding, so flying does not stop it.
- **Gamepad rows** apply to every gamepad assigned to the player. On an assigned controller without a gamepad layout, D-pad rows use its directional pad or POV hat (diagonals count for both directions).
- **Device rows** belong to one device: Rewired's persistent device GUID, or a unique hardware match when there is no GUID. A disconnected device keeps its rows, shown as "(not connected)", and they work again when it reconnects. Identical devices that cannot be told apart stay inactive. A device row on a joystick replaces the gamepad rows of the same binding on that joystick.
- **Release first.** After a device change, a settings change, a blocked moment or a frame in which your mod did not read the binding, a held control must be released before it fires again.
- **Log.** The first time each control fires, your mod logs `Binding '<name>' fired from <control>`. When a binding is first read and when controllers change, it logs the controllers it can see (at most 8 times). Players can send you this when a control does not work.

## Use in a mod

Add the package:

```xml
<PackageReference Include="NuclearOption.Bindings" Version="0.1.1" />
```

The `.nupkg` is also attached to each [GitHub release](https://github.com/scramd/nuclear-option-bindings/releases) for a local package folder. Without NuGet, copy `src/` into your repository and import `src/NuclearBindings.props` from your `.csproj`.

Your mod must reference these game assemblies from `NuclearOption_Data/Managed` (most mods already do): `Assembly-CSharp`, `Assembly-CSharp-firstpass`, `Rewired_Core`, `UnityEngine`, `UnityEngine.CoreModule`, `UnityEngine.InputLegacyModule`, `UnityEngine.IMGUIModule`, `UnityEngine.UI`, `UnityEngine.UIModule` and `Unity.TextMeshPro`, plus `BepInEx`. Do not ship them with your mod.

```csharp
using NuclearBindings;

private InputBinding toggle = null!;

private void Awake()
{
    toggle = InputBinding.Bind(Config, Logger, "Controls", "Toggle Page", "F8", "Switches the page.");
}

private void Update()
{
    if (toggle.IsDown()) page = !page;
}
```

Read a binding every frame in which it matters, from your mod's own `Update` or `LateUpdate`. Do not create a separate `DontDestroyOnLoad` object for it.

| Member | Use |
| --- | --- |
| `InputBinding.Bind(config, log, section, key, defaultText, description, allowFullMap = false)` | Creates the setting and its F1 drawer. `defaultText` uses the row format: `""` (unbound), `"F10"`, `"Ctrl+F9"`, `"Gamepad:DPadUp"`, or several rows separated by `;`. `allowFullMap: true` keeps it active while the full map is open. |
| `IsDown()` | True on the frame the binding goes from released to held. |
| `IsHeld()` | True while any bound control is held. |
| `ToString()` | The bound controls by name, for your messages, or "Not bound". |
| `Rows`, `LogFired(source)` | The parsed rows, for mods that need their own per-control logic (tap or hold, double presses). Call `LogFired` when such logic acts. |
| `RowNote` | Optional text shown under a row in F1. |
| `Capturing`, `CancelCapture()` | Whether F1 is listening for a control, and stop it. |
| `InputContext.ExtraBlock` | Optional `Func<bool, bool>` that blocks all bindings of your mod; it gets the binding's `allowFullMap` value. |

Gamepad control names: `DPadUp`, `DPadDown`, `DPadLeft`, `DPadRight`, `ButtonSouth`, `ButtonEast`, `ButtonWest`, `ButtonNorth`, `LeftShoulder`, `RightShoulder`, `LeftTrigger`, `RightTrigger`, `LeftStickButton`, `RightStickButton`, `Select`, `Start`.

### Moving from `KeyboardShortcut`

Keep the setting's section and key, and change the setting to `InputBinding.Bind`. Saved values convert in place the first time the mod loads:

- `F7 + LeftControl` becomes `Ctrl+F7`.
- `None` or empty becomes unbound.
- `JoystickButton0` to `JoystickButton9` become gamepad controls in Unity's XInput order: A, B, X, Y, LB, RB, Back, Start, LS, RS.

To move a value from a setting that you remove, read it with `BindingMigration.TakeLegacy(config, section, key)` (it returns null when it was never saved, and removes the old key) before you bind the new setting. `BindingMigration.IsSaved` tells whether the new setting is already in the file. Then call `config.Save()` once.

### Other Rewired games

Everything except one check is independent of Nuclear Option: the game's menu, chat, loading and map state. Define `NUCLEAR_BINDINGS_GENERIC` to leave that check out. Then `Assembly-CSharp` is not needed, and you supply your game's conditions through `InputContext.ExtraBlock`.

## Checks

```sh
dotnet run --project tests/BindingChecks -c Release
```

These logic checks cover the row format, the conversions from `KeyboardShortcut` text and real BepInEx configuration files. They need no game files, and CI runs them on every push. `tests/CompileCheck` compiles every file against your own game install (see the comment in its project file); CI cannot run it. Neither check proves that a press reaches a mod in the game. The fired log line shows that.

## License

MIT. See [LICENSE](LICENSE).
