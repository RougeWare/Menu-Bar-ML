# Menu Bar ML
An XML-based markup language for cross-platform menu bars

# Idea
This is currently just an idea, but I like how promising it's looking. It's inspired by Android's, JavaFX's, iOS's, and Mac OS's XML layouts, but is not platform-specific.

Consider this version **0.0.1 &lambda;**

## Sample
Here's the sample code I'll be referencing throughout this explanation. If I change my mind, I'll change this code, and the explanation below.
```XML
<?xml version="1.0" encoding="utf-8"?>
<menu type="bar">
  <menu id="APP" title="$APP_MENU_NAME">
    <group>
      <item id="ABOUT" title="About $APP_NAME" />
      <item id="SETTINGS" title="Settings" title-osx="Preferences" hasFurtherActions />
    </group>
    <group>
      <item id="QUIT" title="Quit $APP_NAME" />
    </group>
  </menu>
  <menu id="EDIT" title="Edit">
    <group>
      <item id="CUT" shortcut="Ctrl+X" shortcut-osx="Cmd+X" title="Cut" />
      <item id="COPY" shortcut="Ctrl+C" shortcut-osx="Cmd+C" title="Copy" />
      <item id="PASTE" shortcut="Ctrl+V" shortcut-osx="Cmd+V" title="Paste" />
    </group>
    <group>
      <menu id="FIND" title="Find">
        <item id="FIND" shortcut="Ctrl+F" shortcut-osx="Cmd+F" hasFurtherActions title="Find" />
        <item id="FIND_REPLACE" shortcut="Ctrl+H" shortcut-osx="Cmd+Opt+F" hasFurtherActions title="Find and Replace" />
      </menu>
    <group>
  </menu>
</menu>
```

## Elements
### `<menu>`
This represents any menu or submenu, including the menu bar.

- The menu bar should always be the singular `<menu>` element that wraps all other elements. For completeness, it should be given a `type="bar"`, but implementations can choose to ignore this attribute, especially for systems where menu bars within menus are not possible. Being a singular container element should suffice to denote it as the menu bar.
- Non-bar menus are _required_ to have a `title` attribute, which specifies the text on the menu's label. The value of his attribute is plain Unicode text (`/^.+$/`).
  - The whole or part of the title can be optionally replaced at runtime by using an identifier matching the pattern `/\$(\w+)/`.
- Non-bar menus are _encouraged_ to have an `id` attribute, which is an identifier used to identify the menu within its parents. These can be recursively concatenated with the `id`s of its parents separated by periods (`.`) to make an ID which is unique within the file. For instance, in the above example, the Find menu is given the `FIND` `id`, which would allow it to be identified as `EDIT.FIND`. The value of his attribute must only be word characters (`/^\w+$/`).
- Menus which are to be explicitly disabled by default should specify the `disabled` attribute.
- Menus which are to be explicitly not shown by default should specify the `hidden` attribute.
- There is no encouragement for how sub-menus should be displayed. Traditionally, sub-menus are represented like an `<item>` within their parent, with a triangle indicator, and appear as another menu to the side of the parent menu when activated. However, if the you want to be creative with how this can be shown, it is highly encouraged, especially for small screens! For instance, a sub-menu might be permanently shown like a `<group>` if it has few `<item>`s, or might replace the parent menu, etc.

### `<item>`
An item is an endpoint action item.

- Items are _required_ to have a `title` attribute, which specifies the text on the item's label. This works just like `title`s on `<menu>`s.
- Items are _encouraged_ to have an `id` attribute, which works just like that on a `menu`. For instance, in the above example, the Find item is given the `FIND` ID, which would allow it to be identified as `EDIT.FIND.FIND`.
- Items which provide more actions after selected (e.g. pop up a dialog) are _encouraged_ to have a `hasFurtherActions` attribute. The UI should indicate this, usually with an ellipsis.
- Items which are to be explicitly disabled by default should specify the `disabled` attribute.
- Items which are to be explicitly not shown by default should specify the `hidden` attribute.
- Items are _encouraged_ to have a `shortcut` attribute, which allows you to specify a keyboard shortcut which will activate the item. This is a DSL which is elaborated upon [below](user-content-shortcut-value-dsl).
  - To allow for more familiarity for users across many environments, items can optionally specify alternate shortcuts for specific operating systems using one of the following, where more specific and newer ones override less specific and older ones. Others not specified here may be used if necessary, but their usage would be proprietary or de-facto, so beware when using them as they may be defined in future versions of this spec.
    - `shortcut-win` for any version of Windows
      - `shortcut-win-9x` for Windows 95 and 98
      - `shortcut-win-xp` for Windows XP+
      - `shortcut-win-vst` for Windows Vista+
      - `shortcut-win-phn` for any Windows Phone
      - `shortcut-win-7` for Windows 7+
      - `shortcut-win-7p` for Windows Phone 7+
      - `shortcut-win-8` for Windows 8+
      - `shortcut-win-8p` for Windows Phone 8+
      - `shortcut-win-10` for Windows 10+
      - `shortcut-win-10p` for Windows 10+ for phones
    - `shortcut-unx` for any Unix or Unix-like OS
    - `shortcut-osx` for any version of Mac OS X
    - `shortcut-ios` for any version of iOS
    - `shortcut-and` for any version of Android
    - `shortcut-lnx` for any flavor of Linux desktop environment
      - `shortcut-lnx-gnu` for Linux running Gnome
        - `shortcut-lnx-gnu-1` for Linux running Gnome 1
        - `shortcut-lnx-gnu-2` for Linux running Gnome 2
        - `shortcut-lnx-gnu-3` for Linux running Gnome 3
      - `shortcut-lnx-kde` for Linux running KDE
      - `shortcut-lnx-unt` for Linux running Unity
      - `shortcut-lnx-cnm` for Linux running Cinnamon
      - `shortcut-lnx-bdg` for Linux running Budgie
      - `shortcut-lnx-pnt` for Linux running Pantheon
      - `shortcut-lnx-mat` for Linux running Mate
      - `shortcut-lnx-lxd` for Linux running Lxde
      - `shortcut-lnx-xfc` for Linux running Xfce
    - `shortcut-txt` for text-only interfaces
      - `shortcut-txt-vi` for VI-like environments
      - `shortcut-txt-emc` for EMACS-like environments

### `<group>`
This represents a grouping of items andor menus. Once rendered, if two groups are flush against eachother, a horizontal separation line should be drawn on the UI.

- Groups can have an optional `id` attribute, but this is currently ignored.
- Groups which are to be explicitly not shown by default should specify the `hidden` attribute.

## `shortcut` value DSL
Shortcut values are a DSL which match the following pattern:

- `Shortcut` = `Modifiers` `Key`
- `Key` = Any singular keyboard key which is not a `Modkey`.
- `Modifier` = `Modkey` [`"+"` `Modifier`]
- `Modkey` = (`Cmd`/`Meta`/`Win`) / `Ctrl` / `Alt` / `Shift`
- `Cmd`/`Meta`/`Win` = The Command, Meta, or Windows key, depending on the OS. This should still correspond to the same keyboard key.
- `Ctrl` = The Control key.
- `Alt` = The Alt key.
- `Shift` = The Shift key.
