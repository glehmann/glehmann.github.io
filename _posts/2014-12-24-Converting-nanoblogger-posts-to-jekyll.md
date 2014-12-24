---
title: Converting nanoblogger posts to jekyll
---

So my old posts were done with [nanoblogger], and I'm now using [jekyll].
Fortunately, both are using a quite similar format, so importing my old posts
should be not too difficult.

Even better, there is already [a tool](https://gist.github.com/atomicules/881023#file-nanoblogger2jekyll-rb)
for that. I had to modify it a little though, in order to avoid adding `categories`
and other things that I don't need in the front matter, use `.md` instead of `.html`
as file extension and remove the extra line break at each line.

Here is my modified version:

~~~ruby
# Script to convert a directory of Nanoblogger posts to Jekyll
#
# Nanoblogger is a command line, static blogging app, not that
# dissimilar to Jekyll: http://nanoblogger.sourceforge.net/
# 
# It's been years since I've used it though, but the below script
# worked for me in converting the files to Jekyll. 

Dir['*.txt'].each do |f|
	# Need to read file to find title
	title = ''
	lines = IO.readlines(f)
	lines.each do |l|
		if /TITLE/ =~ l 
			title = l[6..-1].strip # strip leading whitespace, trailing return
			break
		end
	end
	lines.slice!(0..lines.index("BODY:\n")) # Remove Nanoblogger front matter
	begin
		lines.slice!(lines.index("END-----\n")..-1) # Remove Nanoblogger end matter
	rescue
		lines.slice!(lines.index("END-----")..-1) # Might not have line break
	end
	# Add in Jekyll Yaml Front Matter
	# !Important note to self. I had already split the archive of posts into subfolders based
	# on category. So this script would be run on each subfolder and the category manually set 
	# below.
	lines.unshift("---\n", "title: #{title}\n", "---\n")
	# Replace data in file #IO.writelines?
	File.open(f,"w") do |file|
		lines.each do |l|
			file << l
		end
	end
	# Then rename file based on title
	title = title.gsub(/\s/,"-") # replace spaces, any other dodgy characters can manually fix
	title = "-"+title # Add leading -
	newf = f.gsub(/T.*/, title+".md")
	newf = newf.gsub(/\\|\/|:|\*|\?|\"|<|>|\|/, "_") # Windows safe filenames
	File.rename(f, newf)
end
~~~

Then I just had to copy all the `.txt` from [nanoblogger]'s `data` directory to
[jekyll]'s `_posts` directory, run the scripts, and a add few tags where needed.
Done!

[nanoblogger]: http://nanoblogger.sourceforge.net/
[Jekyll]: http://jekyllrb.com/
