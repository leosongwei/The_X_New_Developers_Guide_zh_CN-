The X Client Ecosystem
======================

**Alan Coopersmith**

On both the client and server side of X, there is a large mass of infrastructure that must be mastered to get work done. This chapter covers the client side. Start here to learn how to build, maintain and develop X applications.

The Structure of an X Client Application
----------------------------------------

A typical X client application is built on top of a number of libraries, providing common functionality that many applications need.

<p>
<img src="./img/stairstep.svg" width="80%" />
</p>

Applications typically use a "toolkit" that provides common user interface elements, such as menus, buttons, text fields and other standard widgets. Modern toolkits include Qt, GTK+, and FLTK. Older toolkits, still supported for legacy applications, include the Athena Widgets (Xaw) and Motif (Xm). Some toolkits are cross-platform, which can assist in making your application usable in non-X environments such as Wayland, Android, MacOS X or Microsoft Windows. A list of X toolkits is available at http://en.wikipedia.org/wiki/Listofwidget_toolkits .

Most X applications need to render something. The X community has great library support for efficient, convenient drawing in most environments. These libraries deal with optimizing for specific video hardware, output formats, etc. For 2D graphics, the Cairo library (http://cairographics.org) is commonly used today, both by applications directly and by toolkits for their rendering needs. For 3D graphics, the OpenGL API is the industry standard, implemented in the free software stack by the Mesa (http://mesa3d.org) libraries.

Proper text handling is trickier than most developers realize, especially if they have only been exposed to seven-bit ASCII text. Handling Unicode characters, loading appropriate fonts for different languages, figuring out how to place characters in languages with complex rules for connecting characters or overlaying diacritical marks, even just figuring out whether characters are displayed in left-to-right or right-to-left order: these are all solved problems for existing libraries such as Pango (http://pango.org) and Harfbuzz http://harfbuzz.org. Toolkits use these libraries to handle text layout in the widgets they provide, Cairo uses them for text in canvases it renders, and applications use them for any text they need to place on screen directly.

Underneath all these libraries is X's Xft extension. Xft provides for actually displaying text glyphs on screen, using fontconfig (http://fontconfig.org) to find an appropriate font for each glyph, and Freetype (http://freetype.org) to render each glyph as an image that Xft can send to the X server for display.

The actual communication with the X server is handled by one of two sets of libraries---Xlib or XCB. Each library family provides programmatic access to the actual X protocol requests, hiding the details of X11 protocol connections and encoding/decoding from the clients. Xlib and XCB are covered in more detail in the "Xlib and XCB" chapter of this book.

While it is theoretically possible to write an X client using pure system calls, generating all the X protocol message encoding and decoding yourself, that would usually be a massive waste of time and a source of bugs. It is also possible to write an X client using just the X11 libraries, and to generate all the code for drawing menus, buttons, text yourself, but that also wastes much time reinventing the wheel; multiple programmer-years of effort would be required to get the full functionality required to support internationalization, accessibility, desktop integration, input handling, and many other features provided by toolkits.

Building X client code
----------------------

X.Org, along with many of the open source toolkits and libraries mentioned above, has standardized on the use of the pkg-config system (http://www.freedesktop.org/wiki/Software/pkg-config) for determining the required set of compiler and linker flags to use when building software that uses these libraries.

For example, to find the flags needed to link against the libxcb and libxcb-util libraries, you would run:

```
pkg-config --libs xcb xcb-util -lxcb-util -lxcb
```

You should not simply copy the results into your build scripts. Instead, run pkg-config at build time: the results may vary by platform, by install location (different "-I" or "-L" flags to find the right path), or by version as requirements change.

If your software is built using the GNU autoconf system, pkg-config provides a simple macro you can use for finding the required flags for your build in the configure.ac script. You can also specify minimum required versions of a given library, as shown in this example:

```
PKG_CHECK_MODULES(XCBLIBS, [xcb >= 1.6] xcb-icccm xcb-shape)
```

If all the required libraries are found, with sufficient versions and any required dependencies they specify in their pkg-config files, then autoconf will provide variables in the generated Makefiles named XCBLIBSCFLAGS and XCBLIBSLIBS. These variables will provide the flags you need to pass to the compiling and linking stages of your build.

X libraries
-----------

The following list are the X libraries offering C language API's maintained by X.Org. As described above, these API's provide a lower level of functionality on top of which the higher level libraries from other providers are built. Language binding layers for other languages, such as Python, Perl and Tcl, are also available from various sources.

There are two generations of libraries at the core of the X protocol stack - the newer XCB and the older Xlib. The "Xlib and XCB" chapter in this guide explains the differences, and describes the interoperability between the two families.

Documentation for almost all of these libraries is included with the libraries themselves, in the form of Unix man pages and DocBook/XML reference docs. For those with DocBook docs, the HTML, PDF, and plain text format documentation generated from those docs has been posted online at http://www.x.org/releases/current/doc/ .

###XCB family of protocol and utility libraries

The newer generation of X protocol encoding &amp; decoding libraries are built on top of the XCB core, with the encoding &amp; decoding functions auto-generated from XML descriptions of the protocol. *libxcb provides both the connection management functions, and the handling of the core X protocol, with additional libraries provided for each X extension*:

```
libxcb-composite    libxcb-res
libxcb-damage       libxcb-screensaver
libxcb-dpms         libxcb-shape            libxcb-xinerama
libxcb-dri2         libxcb-shm              libxcb-xinput
libxcb-glx          libxcb-sync         libxcb-xprint
libxcb-randr        libxcb-xevie            libxcb-xtest
libxcb-record       libxcb-xf86dri      libxcb-xv
libxcb-render       libxcb-xfixes       libxcb-xvmc
```

Unfortunately, not all extensions are yet supported by XCB libraries. XKB is a noticeable gap---libxcb-xkb is still being worked on, due to the complex nature of the XKB protocol definition. Two extensions, BigRequests and XC-MISC, are fundamental to the handling of other requests; these extensions are thus built directly into libxcb instead of being provided via separate libraries.

There are also some utility libraries built on top of the XCB protocol libraries to provide common higher-level functions. These libraries are still under development, and are still seeing some changes to their API between versions that may break compatibility.

* libxcb-icccm and libxcb-ewmh: These libraries provide functions to get and set the properties specified in the ICCCM and EWMH standards for interacting with window managers and desktop environments. (See the "Properties" section in the "Concepts" chapter for more details on these.)

* libxcb-image: This library moves images (client-side pixmaps) to and from the X server. In Xlib, XImage and XShmImage provide similar functionality.

* libxcb-render-util: This library provides convenience functions for drawing images and text using the Render extension.

* libxcb-util: This library is a grab-bag of convenience functions and definitions that failed to fit into the others. The library includes standard atom constants, atom caching, event decoding, and display structure operations.

###Xlib family of protocol and utility libraries

The older generation of X protocol handlers is built on top of libX11, a library known colloquially as Xlib. Besides protocol handling, libX11 also includes a large number of utility functions. Xlib provides support for international input methods and for ICCCM property handling. It also provides legacy versions of other functionality (such as color management) now commonly provided in higher level libraries; the modern libraries offer better integration with toolkits and applications. Like the XCB family, many extensions have their own Xlib-based library for handling the requests for that extension. A common set of older extensions is, however, grouped into a single libXext library.

Libraries for individual extensions:

```
libXcomposite   Composite extension
libXdamage      Damage extension
libXevie        XEvIE extension
libXfixes       X-Fixes extension
libXfontcache   X-TrueType font cache extension
libXi       Xinput extension
libXinerama     Xinerama extension
libXp       Xprint extension
libXrandr       X Resize and Rotate extension
libXrender      RENDER extension
libXres     X Resource extension
libXss      MIT-SCREEN-SAVER extension
libXTrap        X Trap extension
libXtst     XTEST extension, Record extension
libXv       Xvideo extension
libXvMC     Xvideo Motion Compensation
libXxf86dga     XFree86 Direct Graphics Access extension
libXxf86misc    XFree86-Misc extension
libXxf86vm      XFree86 Video Mode extension
libdmx      Distributed Multihead X extension
```

Extensions covered by libXext:

```
DBE - Double Buffering Extension
DPMS - Display Power Management Signalling
MIT-MISC - X11R3 Bug Compatibility Mode [obsolete]
AppGroup - Broadway Application Grouping [obsolete]
EVI - Extended Visual Information
Generic Event - X Generic Event extension
LBX - Low Bandwidth X [obsolete]
MultiBuf - Multiple Buffering Extension [obsolete]
SECURITY - X Security model extension
SHAPE - Non-rectangular Shaped Window Extension
MIT-SHM - Shared Memory Images/Pixmap Extension
SYNC - X Synchronization Extension
TOG-CUP - Colormap Utilization Protocol extension [obsolete]
XTestExt1 - X11 Input Synthesis Extension [obsolete]
```

Quite a few of those extensions are no longer in common use, or supported by the X server. However, since libXext bundled them into the same library, they cannot be easily removed without breaking backwards compatibility with existing programs.

###X Toolkit Intrinsics and legacy toolkits

X.Org originally developed the X Toolkit Intrinsics (libXt) to provide common functionality to multiple toolkits. libXt allowed for configuration via a common X resource format, a standardized event loop, and other underlying functionality. libXt was used in many early toolkits, including X.Org's sample Athena Widgets toolkit (libXaw), Sun's OpenLook toolkit (libXol), and the Open Software Foundation's Motif toolkit (libXm). Modern toolkits however have mostly eschewed libXt. These toolkits use some combination of other common layers (glib for event loops, for instance) and their own toolkit-specific desktop-specific library code. Thus, libXt is maintained mainly for use by legacy applications.

The Athena Widgets toolkit is the toolkit used by many X.Org sample applications, as well as some applications from other sources. The original version (libXaw) has a very simple 2-D look, while the enhanced fork (libXaw3d) updates the look with a more 3-D feel. Neither version is actively developed. The Athena Widgets, like the underlying libXt, are mainly maintained for existing applications. Writing new applications against Athena Widgets is discouraged. The toolkit lacks much of the functionality needed for proper operation of modern applications. For example, internationalization support is minimal, as is support for accessibility technologies such as screen readers or alternative input devices for users with physical disabilities. It would also be awkward to try to use the Athena Widgets on non-traditional computing devices such as cell phones or tablets.

Both libXt and the Athena libraries are built on top of the Xlib family of protocol handling libraries.

###Libraries for related protocols

While the X11 protocol and it's extensions are the core of the X Window System, they are not the only protocols used in X, and additional libraries are provided for software that needs to operate over these protocols.

* libFS handles the encoding and decoding of the X Font Service protocol, allowing X servers or other programs to retrieve fonts from a remote XFS font server. This is purely legacy functionality, and should not be used in modern client or server configurations.

* libICE supports communication between multiple clients using the Inter-Client Exchange protocol.

* libSM covers the X Session Management protocol, built on top of libICE. libSM provides desktop environments a mechanism to save the state of running clients before stopping an X session, and to restore this state at next login.

* libXdmcp provides the encoding and decoding for the X Display Manager Control Protocol. XDMCP provides a poorly-resourced local X server (typically an X terminal) with the ability to authenticate clients on a remote computer.

###Utility libraries

X.Org provides a handful of utility libraries, providing convenient encapsulation of functions needed by multiple clients. Many of these are based on the Xlib protocol library stack.

* libXau manipulates .Xauthority files, used by xauth, X servers, and display managers to store shared secret data such as MIT-MAGIC-COOKIEs used for authenticating X clients attempting to connect to an X server. libXau is used by both Xlib and XCB.

* libXcursor handles the alpha blended and animated cursors provided by the Render and XFixes extensions, including file formats for storing and reading them from disk. libXau is used by both Xlib and XCB. [iirc --po8]

* libXfont is the implementation of the legacy core protocol font rendering mechanism. libXfont is used by X servers and the XFS font server to load and render fonts in a variety of formats. The rendering code for some of these formats is built into the library. Other formats are passed to the FreeType library to rasterize. Some of the X.Org font utilities use libXfont to manage font files and metadata files such as fonts.dir. Normal clients will not link to libXfont directly, but access the functionality via X protocol requests. Modern clients avoid using X core fonts entirely, relying on fontconfig / libXft or on libraries utilizing these tools. libXfont thus represents strictly legacy functionality.

* libfontenc handles the encoding tables used to map fonts across character encodings such as ISO-8859 and Unicode. Like libXfont, libfontenc is legacy software used by the X.Org core font infrastructure software rather than by normal clients.

* libXft is a font library that clients use directly. It cooperates with fontconfig to find fonts, with Freetype to render them, and with either libXrender or libX11 to draw the rendered text on screen or into a pixmap. Some clients use libXft directly, but most use higher-level libraries such as pango or API's provided by the application toolkit. These API's handle text shaping and layout rules to determine proper placement and connection of glyphs in complex languages.

* libXmu and libXmuu are a collection of miscellaneous utilities used by the X.Org sample clients. libXmuu consists of utilities that build upon just the core libX11 libraries. libXmu depends on the Athena Widgets toolkit, pulling in libXaw and libXt.

* libXpm supports the XPM XPixMap image file format. XPM represents image data in a portable ASCII text format. This was especially important in legacy X applications, which tended to have XBM or XPM images compiled right into the C code. Modern applications use external libraries or toolkit functionality to access image files.

* libxtrans is the shared code implementation of the transport layer (sockets, pipes, etc.) used for communications by a number of the protocol implementations in the X Window System, such as libX11, libICE, libFS, xfs, and the X server. It is not actually a shared library like the others, but shared C code which is built into each library or program. The XCB library stack has its own connection management code in libxcb and does not use xtrans. Indeed, XCB replaces one of several instantiations of libxtrans in modern Xlib.
