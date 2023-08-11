# Adding music to a PiJu server

Before you can add music to the PiJu system, you need to create a destination
directory for it first. See Step 1, below. Once that's done, you have two options
for adding music to PiJu: either getting the Pi to pull music from a source, or
to push music onto the Pi. Either way, you need to

## Step 1: Create a destination directory

Logged in as piju on the pi:

```sh
mkdir music
```

## Step 2a: Copying from another computer/server

Logged in as piju:

```sh
cd music
scp -r <SOMEWHERE> .
```

Obviously, it's up to you to figure out where your music is coming from. Or,
you may choose to scp/rsync *to* piju. Either way, now's a good time to get a
drink: transferring your music library to the Pi will probably take a while.

## Step 2b: Pushing from a computer to the Pi

It might be better to use `rsync`, rather than scp, to make it faster to
add further music to PiJu if/when you add more music to your local computer.

From a Mac:

```sh
rsync -rvts /Users/fred/Music/ piju@piju_address:music
```
