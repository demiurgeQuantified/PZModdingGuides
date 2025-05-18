The decompiled source files from [ZomboidDecompiler](https://github.com/demiurgeQuantified/ZomboidDecompiler) can be used to remotely debug the game. This allows you to set breakpoints in the game's engine, analyse state, and anything else debugging ordinarily allows. To set up a project for remote debugging, follow these steps:

1) Decompile the game using ZomboidDecompiler with the `--remap-line-numbers` option specified. The easiest way to do this is to open `cmd` in the same directory as ZomboidDecompiler and run `ZomboidDecompiler.bat --remap-line-numbers`.
2) After creating your project in your IDE with the output, you need to be able to attach it to the running game. Instructions are provided for IntelliJ IDEA only, as that's the only IDE I use for java. You can probably find instructions on how to set up remote debugging for any IDE on google: nothing here is specific to Zomboid.
   1) Open the Run/Debug Configurations menu.  
      ![image](https://github.com/user-attachments/assets/1514ab6b-4625-4844-bb0f-b9617f0f12b9)
   2) Create a new Remote JVM Debug configuration.  
      ![image](https://github.com/user-attachments/assets/0a60ae5e-af84-4609-b725-db028adf377c)
   3) Name it something like 'Attach to PZ'. The default settings should be correct, but, just in case, make sure they match the highlighted settings in the image.  
      ![image](https://github.com/user-attachments/assets/dae9f8bc-ff78-49ba-89a2-6f55675f5271)  
   Your project is now correctly set up for debugging!
 3) To be debugged, the game itself needs to host a remote debugging server. You can do this by adding these JVM arguments to the start of your game's launch options: `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005`. JVM arguments have to be at the start of the launch options, before any game arguments (such as `-debug`), and you must add `--` after all JVM arguments to mark the end.
   - For most users, this means the final launch options will be:  
     `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 -- -debug`  
     This enables remote debugging, as well as the game's normal debug mode.
5) After launching the game, attach your debugger. If you're using IntelliJ IDEA, you can do this by pressing the green debug button while your debugging configuration is selected.  
   ![image](https://github.com/user-attachments/assets/0a8331cb-91ed-40a9-8b8c-aed618cb0140)

Once you've done this, you can use your IDE's debugging facilities as normal. Set breakpoints, analyse the stack, enjoy modding with far more ease than usual! If you only have one monitor, I recommend setting the game to windowed, as in fullscreen (even borderless) you may get trapped inside when the game freezes due to a breakpoint being hit.
