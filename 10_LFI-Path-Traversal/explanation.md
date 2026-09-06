## Chapter — LFI / Path Traversal
#lfi #path-traversal #information-disclosure

The `/projects/download` endpoint takes a `file` parameter and sends back that file. It builds the file path from my input without cleaning it. So if I put `../` in the name, I can go UP out of the intended folder and read other files on the server. This is Path Traversal (a type of Local File Inclusion).

**Recon first:** the `/backup` page leaks backup settings in its response headers, including:
`x-backup-exclude: data/private_notes.txt`
So the server itself told me the name of a sensitive file: `private_notes.txt`.

**Exploit:** I asked the download endpoint for that file using `../` to climb out of its folder:
`GET /projects/download?file=../private_notes.txt`

The server returned the file. Inside, the dev even wrote that this endpoint "joins paths without sanitizing" — and the flag was there:
	`🚩🚩🚩 FLAG{d0t_d0t_sl4sh_4ll_th3_w4y_d0wn}`


### Impact
An attacker can read files outside the allowed folder — config files, secrets, source code, or private notes. Here it leaked a private notes file with a flag, but the same trick could read much more sensitive files on the server.

### Prevention
- Do not build file paths from user input. If you must, resolve the final path and check it stays inside the allowed folder (reject anything with `../`).
- Better: serve files by an ID from a fixed list, not by a filename the user controls.