
NOTE: You should agree with the licensing agreement (LICENSE.pdf) before using the tool. 

A. Directory orgnization:
    ./check                # the lisp files for ddtbd
    ./ddtbd                # the plugin for spectre detection
    ./toy                  # a toy example from Spectre paper: https://spectreattack.com/spectre.pdf
    ./tool                 # a tool to profile the output (incidents) of the detection 
    ./testcases       	   # simple test cases
        - Kocher_tests/    # the examples from Paul Kocher's post: https://www.paulkocher.com/doc/MicrosoftCompilerSpectreMitigation.html
        - tests            # some examples written by us
        - negtives/        # the negtive examples which will not be detected. 
        

B. How to install and run:

1. Install opam and Bap.
    Please follow the instructions on the following page to install opam and bap:

    A. Install opam-1.2.2 or later.
        ## $ sudo add-apt-repository --yes ppa:avsm/ppa
        ## $ sudo apt-get update
        ## $ sudo apt-get --yes install opam

    B. Initialize opam and to install OCaml compiler.
        ## $ opam init --comp=4.05.0
        ## $ eval `opam config env`

    C. Install bap and its system dependencies
        ## $ opam depext --install bap


    *Reference: https://github.com/BinaryAnalysisPlatform/bap/wiki/Build-tips-and-tricks 


2. Install and compile devlopment version of Bap.
    clone bap project:
    ## $ git clone https://github.com/BinaryAnalysisPlatform/bap
    
    pin latest bap to opam:
    ## $ opam pin add bap to/your/bap/project/path

    opam will automatically compiles lastest bap.

    update your PATH:
    ## $ eval `opam config env`

    Make sure bap is the latest version
    ## $ bap --version 
    ## 1.5.0-dev


3. Copy "check/" directory to your opam share directory.
   ## $ copy check -r ~/.opam/4.05.0/share/bap
   NOTE: This path maybe differnt according to your opam installation and opam switch


4. Build and install ddtbd plugin.

    Install plugin:
    ## $ bapbundle install ddtbd.plugin


5. Run the toy example. 
    ## $ bap test/test --recipe=check


6. Profile the output of detection.
    ## objdump -S test > test.asm
    ## ./tool/incidents_profile.py incidents test.asm

    You can find a profile file with name "incidents_profile.txt" in you directory. 

    The content of incidents_profile.txt
	====================================
	@branches: 12                # all condition branches
	@S1: 1 (8.333%)              # tainted branches <CB>
	@S2: 1 (8.333%)              # tainted branches with IM1 <CB, IM1>
	@S2_avg_dis: 10              # average distance between CB and IM1
	@S3: 1 (8.333%)              # tainted branches with IM1, IM2 <CB, IM1, IM2>
	@S3_avg_dis: 7               # average distance between CB and IM2

	S1#4005c8                    # address of CB

	S2#4005c8#4005cf#3           # address of CB and IM1 along with the distance of CB and IM1
	S2#4005c8#4005de#7

	S3#4005c8#4005cf#4005de#7    # address of CB, IM1, IM2 along with the distance of CB and IM2

	taint#400648                 # tainted addresses. 
	taint#4005d9
	taint#4005cf
	taint#4005bd
	taint#400642
	taint#4005d6
	...
	===================================


C. Testing for Paul Kocher' examples:
    ## $ cd Kocher_tests/v01
    ## $ gcc test.c -g -o test
    ## $ bap test/test --recipe=check
	## $ ../../tool/incidents_profile.py incidents test.asm


D. Other options
	use $bap --ddtbd-help for more options
	Note: 
        A. Use '--ddtbd-ignore-program-dependencies' or '--ddtbd-ignore-program-dependencies --ddtbd-ignore-control-dependencies' option will give you less detection results, but it may miss some true positives. 
        B. You can edit the "recipe.scm" to enable or disable the options. 


