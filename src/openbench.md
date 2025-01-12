# OpenBench

[OpenBench](https://github.com/AndyGrant/OpenBench) is a distributed chess engine testing framework.
It is more complex than `fastchess` and `cutechess` because it handles more than _just_ running the SPRTs.
While this may seem like overkill, the extra features of OpenBench are incredibly useful and you will be glad to be able to make use of them.

OpenBench has a [wiki](https://github.com/AndyGrant/OpenBench/wiki) that will contain more detailed information than what I will cover here.
I intend to cover the basics on how to set up OB, configure your engine, connect workers, and run SPRTs.
You should refer to that wiki if you have additional questions.

Additionally, we will be setting up OpenBench through a service called [PythonAnywhere](https://www.pythonanywhere.com/).
PA is a good choice for hosting the server as it is free, allows you to keep your OpenBench instance running 24/7, and makes it easy for people outside of your network to see your tests and connect new workers.
Using PA also saves a lot of trouble when dealing with `anaconda` and other python-related tools on your own hardware.

> Many people use OpenBench collaboratively.
> In a shared instance, people tend to leave their worker(s) connected 24/7, executing workloads for any tests that get created.
> Sharing an instance allows you to share resources and (hopefully) have everyone's tests finish faster.
>
> If you are interested in sharing your instance or joining an existing shared instance, ask around, such as in the [OpenBench Discord Server](https://discord.gg/9MVg7fBTpM).

## Overview

OpenBench is a two-part piece of software.
The server provides a UI for creating and viewing tests, as well as other features such as account management.
The client is ran on a "worker" machine and is what actually _runs_ the SPRT in [batched workloads](https://github.com/AndyGrant/OpenBench/wiki/Workload-Assignment).
Multiple clients can connect to the server, meaning you can amass a chess-playing botnet to speed up your tests (assuming you have a few old laptops laying around, or a [GCP](https://cloud.google.com/) account).

If this is your first time setting up OpenBench and running SPRTs, the computer you are using right now will likely be the first worker you connect to your server.

## Setup

As stated before, there already exists a [wiki](https://github.com/AndyGrant/OpenBench/wiki/Creating-Your-OpenBench-Instance) containing much of this information, so refer to that if you get stuck.

### Repository Setup

First, we're going to set up _your copy_ of an OpenBench repository.

1. [Fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) the original [OpenBench repository](https://github.com/AndyGrant/OpenBench), hosted by [AndyGrant](https://github.com/AndyGrant).
    - All modifications you make will be done to _your fork_ of OpenBench.
2. In [`OpenBench/Config/config.json`](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Config/config.json):
    1. Replace the value of [`client_repo_url`](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Config/config.json#L3) with the URL to _your fork_ of OpenBench.
        - Failing to do this _will_ cause your OB instance to [break unexpectedly](https://github.com/AndyGrant/OpenBench/wiki/Creating-Your-OpenBench-Instance).
    1. Add the name of your engine to the [`engines` list](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Config/config.json#L26)
3. In [`OpenBench/Engines`](https://github.com/AndyGrant/OpenBench/tree/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines), create a config file for your engine:
    1. Copy an [existing config file](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines/Stormphrax.json)
    2. Update the value of the [`source`](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines/Stormphrax.json#L4) field to point to the GitHub repository containing your engine.
        - Only GitHub repositories are supported. If you are hosting your engine on another platform, consider making a [mirror](https://docs.github.com/en/repositories/creating-and-managing-repositories/duplicating-a-repository).
    3. Update the value of the [`nps`](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines/Stormphrax.json#L3) field to be the value outputted by your engine's `bench` command.
        - You will likely need to update this in the future- _especially_ if your engine is the aforementioned random-mover baseline.
    4. Update the [`build`](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines/Stormphrax.json#L6) section to be appropriate for your engine.
        - For most people, this will just involve changing the `compilers` value to include whatever compiler(s) your engine needs (i.e. `cargo>=1.83`).
        - Look at other engine config files for references on how to include your engine's dependencies.
    5. (Optional) Change the [default test bounds](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines/Stormphrax.json#L18) for your engine to be `[0.0, 10.0]`.
    6. (Optional) Remove the [adjudication presets](https://github.com/AndyGrant/OpenBench/blob/a8edd633eb80eb4b379233262827cb5bcd8b24ed/Engines/Stormphrax.json#L20-L21) (set them to `None`), only adding them back once your engine is 1200+ Elo.

### PythonAnywhere Configuration

Next, we're going to start the OpenBench server through PythonAnywhere.

4. [Register a new account on PythonAnywhere](https://www.pythonanywhere.com/registration/register/beginner/) and navigate to the "Dashboard" screen.
5. Navigate to the "Consoles" tab, create a new Bash console, and enter the following commands:
    ```bash
    git clone https://github.com/<YOUR GITHUB USERNAME>/OpenBench
    cd OpenBench
    pip3 install -r requirements.txt
    python3 manage.py makemigrations OpenBench
    python3 manage.py migrate
    python3 manage.py createsuperuser
    ```
6. Back at the PythonAnywhere dashboard, navigate to the "Web" tab
    1. Click "Add a new web app"
    2. Choose a name (a common pattern is `<YOUR USERNAME>.pythonanywhere.com`)
    3. Click "Manual configuration (including virtualenvs)" and choose `Python 3.10`
    4. Let it load, then scroll down to the "Code" section.
    5. For the `Source code` section, enter the URL to your OpenBench fork (`https://github.com/<YOUR GITHUB USERNAME>/OpenBench`).
    6. Click on the value for `WSGI configuration file`.
    7. Scroll down until you see a section title `DJANGO`, around line 74.
    8. Delete everything else in this file, leaving only the `DJANGO` section, and uncomment it.
    9. Update the `path` value so that it points to your OpenBench fork.
    10. Replace `mysite.settings` with `OpenSite.settings`.
    11. In the end, the file should contain _only_ the following:
    ```py
    # +++++++++++ DJANGO +++++++++++
    # To use your own django app use code like this:
    import os
    import sys
    # assuming your django settings file is at '/home/<YOUR USERNAME>/mysite/mysite/settings.py'
    # and your manage.py is is at '/home/<YOUR USERNAME>/mysite/manage.py'
    path = '/home/<YOUR GITHUB USERNAME>/OpenBench'
    if path not in sys.path:
        sys.path.append(path)
    os.environ['DJANGO_SETTINGS_MODULE'] = 'OpenSite.settings'
    # then:
    from django.core.wsgi import get_wsgi_application
    application = get_wsgi_application()
    ```
    12. Save the file.

### Instance Configuration

Now we can log into the server and set up your account.

7. Navigate back to the "Web" tab and click the green "Reload" button.
8. You should now be able to navigate to `<YOUR USERNAME>.pythonanywhere.com` and see OpenBench running.
9. On your OpenBench instance, click "Register" on the left sidebar and create an account.
    1. This will be your user account, _not_ the administrative account.
    2. After creating, you should see `Account has not been enabled. Contact an Administrator`.
10. Go to the administration page for your OpenBench instance, located at `<YOUR USERNAME>.pythonanywhere.com/admin`.
    1. Under the "Authentication and Authorization" list, click "Users"
        1. Click on the username of the account you just created, _not_ the `admin` account.
        2. Under "Permissions", check the boxes labeled `Staff Status` and `Superuser status`
            - If other people join your OB instance in the future, you do _not_ need to mark them as Staff/Superusers
        3. Also under "Permissions", (specifically "Available user permissions"), select all permissions that start with `OpenBench` (like `OpenBench | engine | Can change engine`) and add them to your user.
    2. Save your changes and navigate to the "Profiles" tab, on the left sidebar, under the "OpenBench" list .
        1. Check the boxes labeled `Enabled` and `Approver`.
        2. (Optional) Add the name/URL for your engine in the "Repos" field, and your engine's name in the "Engine" field.
11. Navigate back to your OpenBench instance (_not_ the `/admin` panel) and sign out.
12. Sign in with the non-admin account (the one whose permissions you just updated).
13. You should now be able to create a new test by clicking the "+ Create Test" button on the left sidebar, underneath the "Actions" section.
    - Ensure that you can select your engine from the drop-down menus for `Dev Engine` and `Base Engine`, and that the respective `source` fields point to your engine's GitHub repository.
    - Refer to the [wiki page for creating tests](https://github.com/AndyGrant/OpenBench/wiki/SPRT-and-Fixed%E2%80%90Game-Workloads) for explanations on the all of the individual fields.
    - For now, fill in the `Dev/Base Branch` fields with the name of the branch you wish to test (probably `main`, for both, since you're going to be testing the random mover against itself).
    - Fill in the `Dev/Base Bench` fields with the `nodes` value outputted from your engine's `bench` command.
    - (Optional) Under "Test Settings", set `Bounds` to `[-10.0, 0.0]` so this test runs as a non-regression test.
    - Click `Create Engine Test` at the bottom of the page.

You should be redirected back to the index, and should see your test appear on the list of active tests.

### Connecting a Worker

You now need to connect a worker to your instance so that the test can be ran.

As stated before, your first worker will probably be the machine you're using right now, but the steps for setting up a worker are the same for additional machines you connect later.

1. Ensure that the worker machine has the appropriate compilers necessary to run the engine(s) on your instance.
    - Here is a generic command for Ubuntu to install compilers for C/C++/Rust engines:
    ```bash
    sudo apt-get update && sudo apt-get install git python3 pip make g++ gcc cargo cmake -y
    ```
2. Clone _your fork_ of OpenBench:
    ```bash
    git clone https://github.com/<YOUR GITHUB USERNAME>/OpenBench
    ```
3. Navigate to the `OpenBench/Client` directory and install the packages required to run a worker:
    ```bash
    cd OpenBench/Client
    pip install -r requirements.txt
    ```
4. Run the following command to connect the current machine to your OpenBench instance as a worker, replacing the username, password, and URLs appropriately:
    ```bash
    python3 client.py -U <YOUR USERNAME> -P <YOUR PASSWORD> -S http://<YOUR USERNAME>.pythonanywhere.com -T $(nproc) -I $(uname -n) -N 1
    ```
    - If you do not have `nproc` or `uname` on your system, substitute them as follows:
        - Replace `$(nproc) with the number of cores/threads on your system. If this machine needs to do other things _besides_ play chess 24/7, consider using one or two fewer than the number of cores on your machine.
        - Replace `$(uname -n)` with the name you want this worker to have, visible on the `/machines/` page for your OB instance. You can also omit this (including the `-I`) entirely, if you don't care about your workers having names.

Give it a moment to process everything and you should see an output similar to the following:

```
Looking for Make... [v4.3]
Looking for Syzygy... [0-Man]

Scanning for Compilers...
4ku              | g++      (11.4.0)
BlackMarlin      | cargo    (1.83.0)
BoyChesser       | Missing ['dotnet>=7.0.0']
ByteKnight       | cargo    (1.83.0)
Dog              | clang++-14 (14.0.0)
Ethereal         | clang    (14.0.0)
FabChess         | cargo    (1.83.0)
Igel             | g++      (11.4.0)
Obsidian         | gcc      (11.4.0)
Pawnocchio       | zig      (0.13.0)
Polaris          | clang++  (14.0.0)
Stash            | gcc      (11.4.0)
Stockfish        | g++      (11.4.0)
Stormphrax       | clang++  (14.0.0)
Tantabus         | cargo    (1.83.0)
Toad             | cargo    (1.83.0)
Viridithas       | cargo    (1.83.0)
Yukari           | Missing ['cargo>=1.82.0-nightly']
bannou           | zig      (0.13.0)
ice4             | g++      (11.4.0)

Scanning for Private Tokens...

Scanning for CPU Flags...
Found   | POPCNT BMI2 SSSE3 SSE41 SSE42 SSE4A AVX AVX2 FMA
Missing | AVX512VNNI AVX512BW AVX512DQ AVX512F

Requesting Workload from Server...
Workload [Pawnocchio] debug vs [Pawnocchio] debug
```

The `Workload [ENGINE] vs [ENGINE]` line indicates that this worker has received a workload from the server and it will begin executing it.

Navigate back to your OpenBench instance (`https://<YOUR USERNAME>.pythonanywhere.com`) and you should see something like `Active : 1 Machines / 12 Threads / 15.2 MNPS` at the top.
Refresh the page periodically and you should see the test results being updated.
