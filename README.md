# I just ran a rebase and messed up bad. How can I 

To get our repository into a bad state execute the following steps: `in powershell`

## Declare a helper functions

	# Helper function to dump a string to a file
	function writeStringToFile($string, $fileName) {
		$string | Out-File $fileName -Encoding ASCII
	}
	
	# Helper function to give a 'pretty' graph of our git commits
	function gitlog() {
		git log --graph --full-history --all --color --pretty=format:"%x1b[31m%h%x09%x1b[32m%d%x1b[0m%x20%s"
	}

## Setup our git repo

	mkdir GitRebaseReflogSample
	cd GitRebaseReflogSample
	git init
	$ignoreText = "# ignore vim backup files`n*.orig`n`n# ignore git .orig files created during a merge/rebase`n*~"
	writeStringToFile $ignoreText ".gitignore"
	git add .
	git commit -m "added .gitignore file"

## Create a blank file we will use for creating a merge conflict in.

	writeStringToFile "" "file.md"
	git add .
	git commit -m "added empty file.md"

## In a different branch - place text into the file

````powershell
git checkout -b pirate
	
$pirate = "# ye ol' pirate ipsum
`n
Hail-shot clap of thunder line measured fer yer chains keel tack driver fluke case shot sutler. Hang the jib black jack deadlights coffer bilge rat chandler execution dock tackle rope's end Barbary Coast. Scourge of the seven seas fire in the hole interloper smartly black jack tender brigantine Arr hang the jib poop deck.
`n
Broadside provost gangway bring a spring upon her cable fire in the hole heave down knave execution dock overhaul cable. Reef rigging rutters hornswaggle scallywag shrouds gaff handsomely spyglass mutiny. Overhaul fire ship lass furl nipperkin lugger wherry grog blossom spirits mutiny. 
`n
Man-of-war galleon stern Blimey lugger Yellow Jack come about brig gaff scuttle. Grog crow's nest avast Cat o'nine tails jack run a shot across the bow broadside chase cutlass hardtack. Heave to fluke Sink me yawl driver pink cackle fruit take a caulk Pirate Round Letter of Marque.
"

writeStringToFile $pirate "file.md"

git commit . -m "added pirate text"
```

## Now make a different change to the same file in the master branch

````powershell
git checkout master

$lipsum = "# Lorum Ipsum text
`n
Generated from (http://www.lipsum.com)[http://www.lipsum.com]
`n
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam mi libero, fringilla vitae cursus eget, volutpat a massa. Cras at lorem in libero pretium rutrum quis sed nisl. Donec tempus est sit amet nibh consequat vitae pulvinar justo bibendum. Sed in mattis urna. Mauris id nisi metus. Ut porta lectus sed leo ullamcorper eget consequat elit laoreet. Maecenas placerat gravida nibh, ac porttitor leo malesuada sit amet. Pellentesque nisl sapien, blandit eget bibendum at, tincidunt sit amet elit. Vivamus eu lacus leo. Vivamus vitae metus ac nunc congue pretium. Morbi tempor ante vel velit viverra tempus. Nam dapibus diam et eros tincidunt congue. Sed rutrum ultricies laoreet. Nunc faucibus elementum eros, non vestibulum nulla fringilla sed. Aenean turpis enim, ornare et porta sed, porttitor in quam.
`n
Sed vel nunc eu nibh aliquet mollis. Vivamus suscipit, urna non varius lacinia, dolor purus rhoncus ipsum, nec fringilla quam urna ac orci. Proin fringilla tristique ultrices. Pellentesque ac magna nibh. Phasellus magna eros, dignissim ut tristique in, congue sed ipsum. In blandit tincidunt urna, id placerat turpis scelerisque at. Pellentesque nec nisl id metus vehicula pharetra vel malesuada lorem. Vivamus faucibus purus ut leo fringilla posuere. Aliquam adipiscing, erat ac ultricies congue, nunc nulla lacinia magna, nec bibendum purus velit eu neque. Vivamus a nunc id risus tincidunt eleifend quis id lorem.
";

writeStringToFile $lipsum "file.md"
git commit . -m "added lipsum text"
```

## Switch back to the branch and take a look at the commit graph

	git checkout pirate
	gitlog


## Attempt to rebase master against it (you should get a conflict)

	git rebase master

## We should be mid-rebase now with a conflict. Let's simulate a bad merge.

	git add .
	git rebase --continue

## Now if you look at the gitlog you should see a single line of history commits.

	gitlog

#The big question:

* How do we get back to our pre-rebase state so we can re-do the merge (but better)?

This is one case where the git reflog comes in extremely handy.

	git reflog

If you run the `git reflog` command you will see the line where it says `checkout: moving from master to pirate` this is the point in time we would like to restore our state back (just before the rebase happened).

Now copy the sha that is on the same line as that comment and run

	git rebase <sha> --hard

#BAM we're back in business!

Now you can re-run your rebase again and possibly take better care of the merge this time around.

As with much of Git, there are probably other ways around this problem, I'd love to hear if you have a different approach.
