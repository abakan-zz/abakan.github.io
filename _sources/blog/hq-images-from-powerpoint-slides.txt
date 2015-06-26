HQ Images from PowerPoint Slides
================================

.. post:: Jul 16, 2013
   :tags: PowerPoint

I use PowerPoint to collate figures for publications and make :file:`.png` or
:file:`.tiff` files.  Default settings produce output at 96 dpi, but with a
little hack it is possible to save 300 dpi images and satisfy graphics quality
requirements of most journals.

This hack involves adding an entry to Windows Registry and is described in this
Microsoft `KB article`_. Here is a short summary:

1. Click :menuselection:`Start --> Run` (:kbd:`Windows + R`), type ``regedit``
   in :guilabel:`Open` box, and click :guilabel:`OK` or :kbd:`Enter` to start
   :program:`Registry Editor`.

2. Expand the registry to the subkey for the version of PowerPoint that you are
   using:

   :menuselection:`HKEY_CURRENT_USER --> Software --> Microsoft --> Office --> 14.0 --> PowerPoint --> Options`

   You need to choose one of the following version numbers based on Office
   release year:

     ====  =======
     Year  Version
     ====  =======
     2013  15.0
     2010  14.0
     2007  12.0
     2003  11.0
     ====  =======

3. Now, select the :guilabel:`Options` subkey, then :menuselection:`Right Click
   --> New --> DWORD (32-bit) Value`.

4. Name new entry as **ExportBitmapResolution**, and press :guilabel:`Enter`.

5. Select ``ExportBitmapResolution``, then :menuselection:`Right Click -->
   Modify` to change the default resolution.

6. .. rst-class:: strike

      Enter **300** in the :guilabel:`Value` box, select :guilabel:`Decimal`,
      then click :guilabel:`OK`.

   First, select :guilabel:`Decimal` as :guilabel:`Base`, then enter **300** in
   the :guilabel:`Value` box and click :guilabel:`OK`.

7. You can exit :program:`Registry Editor`. You may need to restart PowerPoint
   for changes to become active.

Images that I collate are usually at resolutions higher than 300 dpi, but this
is as good as it gets with PowerPoint. It has helped me save time in several
cases, and I hope you find it useful.

.. _KB article: http://support.microsoft.com/kb/827745


**Edits**:

  * I had copied Office 2013 version number from the `KB article`, but realized
    that it needs to be **15.0** when making these changes on a new virtual
    machine.
  * I changed the order of actions in step **4**, since changing base after
    entering number, changes the value of the number.