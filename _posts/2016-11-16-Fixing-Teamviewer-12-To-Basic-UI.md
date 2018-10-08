---
layout: post
title: Fixing Teamviewer MSI To Allow Machine Install
category: systemadmin
---

## Problem
When trying to install Teamviewer 12 (and probably later versions) through the MSI you can't use a machine installation because the MSI requires a basic UI to be setup.
You can do this with a user install by setting the GPO to basic however that means only an admin can login into the computer and install the MSI

## Solution
Use [Microsoft Orca](https://msdn.microsoft.com/en-us/library/windows/desktop/aa370557(v=vs.85).aspx), in the win32 tools, to add a property in MSI tables to set basic UI

1. Add [UILevel property](https://docs.microsoft.com/en-us/windows/desktop/Msi/uilevel) to the MSI Property table.
	- This allows the user interface level to be set in command line options.
	- You might also be able to use `UILevel=2` in the command line to force to No Ui mode
2. Save that as a mst
3. Add the MST to gpo as a modification
	- Note you have to set the package as advance when creating it in the GPO so you can add the modification


