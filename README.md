Freetype and SharpFont FUSEE Build
==================================

This project contains the sources from the freetype project as well as those from the freetype C# wrapper project, SharpFont.

The goal is to deliver a working SharpFont/freetype package for each platform supported by FUSEE (currently Windows, MacOS and Android).

In addition to the SharpFont project, this project builds the underlying freetype dll from scratch instead of deploying a pre-built dll with the project. The main solution thus contains builds for Android (and iOS in the future) using Visual Studio 15's native code compiling features for mobile platforms. 

Modifications applied to freetype
---------------------------------

 - Updated freetype-2.6.2\builds\windows\vc2010\freetype.vcxproj to Visual Studio 15.
 - Copied project file to FreetypeFusee\FreetypeFusee.Windows\FuseeFreetype.Windows.vcproj
 - Applied SharpFont\Dependencies\freetype2\win64.patch to freetype
 - Changed the following project properties under -> General -> (for ALL CONFIGURATIONS AND ALL PLATFORMS)
    - Configuration Type to "Dynamic Library (.dll) 
    - Changed output directory from `..\..\..\obj\VC2010\...` to `..\..\..\..\bin\freetype\...`
    - Changed Intermediate Directory from `..\..\..\obj\VC2010\...` to `..\..\..\..\tmp\...`
 - Edited ftoption.h: definition of FT_EXPORT:

```C 
	#ifdef PLATFORM_WINDOWS
	#define FT_EXPORT(x)      __declspec(dllexport) x
	#define FT_EXPORT_DEF(x)  x
	#elif PLATFORM_ANDROID
	#endif
```
 
 - Remeved all Configurations (like Multithreaded) besides Debug and Relase 
 - Moved all CLCompile and CLHeader .c and .h entries from FuseeFreetype.Windows.vcproj to FuseeFreetype.Shared.vcxitems besides ftdebug.c which directly addresses Windows.
 - Defined `<FreetypeRoot>..\..\freetype-2.6.2\</FreetypeRoot>` property in both, FuseeFreetype.Windows.vcproj and FuseeFreetype.Shared.vcxitems and replaced all relative paths with $(FreetypeRoot) 
 - Copied ftdebug.c to FreetypeFusee\FreetypeFusee.Windows\
 - Fixed afwarp.h
```C
  FT_LOCAL( void )
  af_warper_compute( AF_Warper      warper,
                     AF_GlyphHints  hints,
                     AF_Dimension   dim,
                     FT_Fixed      *a_scale,
	                 FT_Pos        *a_delta ); // !!! Replaced FT_Fixed with FT_Pos
```
 - Added additional projects referencing the shared FuseeFreetype.Shared.vcxitems sources for Android and iOS.


Modifications applied to SharpFont
----------------------------------
 - FT.Internal.cs: changed `FreetypeDLL`:
```C#
		#if DEBUG 
		private const string FreetypeDll = "freetype262d.dll";
        #else
		private const string FreetypeDll = "freetype262.dll";
        #endif
``` 
 - SharpFont.dll.config: Renamed `freetype6.dll` to self-built dll:
```XML
	<dllmap dll="freetype262.dll">
		<dllentry os="linux" dll="libfreetype.so.6" />
		<dllentry os="osx" dll="/Library/Frameworks/Mono.framework/Libraries/libfreetype.6.dylib" />
	</dllmap>
  <dllmap dll="freetype262d.dll">
    <dllentry os="linux" dll="libfreetype.so.6" />
    <dllentry os="osx" dll="/Library/Frameworks/Mono.framework/Libraries/libfreetype.6.dylib" />
  </dllmap>
```
 - Changed output directory to `{projectroot}\bin\SharpFont\..`.
 - Removed reference to System.Drawing and commented out `using System.Drawing;`, `FTBitmap.ToGdiBitmap()` and `FTBitmap.ToGdiBitmap(Color color)` in FTBitmap.cs.


Modifications applied to Examples
---------------------------------
 - Changed the content items `x86\freetype6.dll` and `x64\freetype6.dll` in the .csproj file to the respective `freetype262.dll`s including debug build distinction (`freetype262d.dll`). 
 - Added speceific x86 and x64 platform builds (in addition to the AnyCPU platform) to explicitly test both platforms on Windows.
 - Changed output directory to `{projectroot}\bin\SharpFontExample\..`.


Usage in FUSEE
--------------
FUSEE uses the libraries built within this project by keeping copies under `{FuseeRoot}\ext\SharpFont\bin\...`.

TODO
----
Realize iOS build using a development MAC paired with VS 2015.


