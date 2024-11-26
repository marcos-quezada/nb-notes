# pnpm link (alvin.codes)

<https://alvin.codes/snippets/pnpm-link>

## Description

It should be easy but it's not.

## Tags

#nodejs #pnpm

## Content

[![Logo with initials](/_astro/logo.CFkCQIM4.svg){height="50px" width="50px"}](/){._home_fs3hf_24}

![](data:image/svg+xml;base64,PHN2ZyBhcmlhLWhpZGRlbj0idHJ1ZSIgdmlld2JveD0iMCAwIDEwMCAxMDAiIGhlaWdodD0iNDAiIHdpZHRoPSIzMCI+IDxwb2x5bGluZSBwb2ludHM9IjEwIDMwLCA5MCAzMCI+PC9wb2x5bGluZT4gPHBvbHlsaW5lIHBvaW50cz0iMTAgNTAsIDkwIDUwIj48L3BvbHlsaW5lPiA8cG9seWxpbmUgcG9pbnRzPSIxMCA3MCwgOTAgNzAiPjwvcG9seWxpbmU+IDwvc3ZnPg==)

-   [Writing](/writing)
-   [Reading](/reading)
-   [Snippets](/snippets){.active}
-   [Speaking](/speaking)
-   [About](/about)

Toggle Theme ![](data:image/svg+xml;base64,PHN2ZyBkYXRhLWFzdHJvLWNpZC1pbzY3emozNSB2aWV3Ym94PSIwIDAgNDUgNDUiIGNsYXNzPSJzdW4iPiA8cGF0aCBkYXRhLWFzdHJvLWNpZC1pbzY3emozNSBkPSJNMjkuNzc0IDIxLjQ1YTguMzcgOC4zNyAwIDExLTE2Ljc0MSAwIDguMzcxIDguMzcxIDAgMDExNi43NDEgME0xNi43NDggNDEuOTkxYTIxLjAzNyAyMS4wMzcgMCAwMS00LjY4NC0xLjY2M2w0LjI2NC04LjYwMmMuODA1LjQgMS42NTguNzAzIDIuNTM1LjlsLTIuMTE1IDkuMzY1bTEzLjU4Ny0xLjQ2NmwtNC4wNzUtOC42OTJhMTEuNDc2IDExLjQ3NiAwIDAwMi4yOS0xLjQyNmw1Ljk5NSA3LjQ5OGEyMS4wOSAyMS4wOSAwIDAxLTQuMjEgMi42Mm0tMjcuODg5LTkuOWEyMC45MjMgMjAuOTIzIDAgMDEtMS42MjItNC42OTZsOS4zODItMi4wMzNjLjE5Ljg3Ny40ODYgMS43MzIuODc4IDIuNTQybC04LjYzOCA0LjE4N20zOS40MzYtNC4yNTNsLTkuMzM1LTIuMjM4Yy4yMDktLjg3MS4zMTUtMS43NzQuMzE1LTIuNjg0bDkuNi0uMDY1di4wNjVjMCAxLjY2My0uMTk1IDMuMzE4LS41OCA0LjkyMm0tMzAuODE4LTkuODY4bC04LjY1Ni00LjE1MWEyMS4wOCAyMS4wOCAwIDAxMi42NTctNC4xODlsNy40NDYgNi4wNmExMS41MDIgMTEuNTAyIDAgMDAtMS40NDcgMi4yOG0xOS4zNi0yLjEyMWExMS41ODUgMTEuNTg1IDAgMDAtMS45MDktMS45MThsNS45NjQtNy41MjJhMjEuMTM0IDIxLjEzNCAwIDAxMy40OTQgMy41MWwtNy41NDkgNS45M20tMTEuNjA4LTQuMDk5TDE2LjY2NC45MjlhMjEuMTYyIDIxLjE2MiAwIDAxNC43NC0uNTM3bC4xODIuMDAxLS4wNzkgOS42LS4xMDMtLjAwMWMtLjg3NSAwLTEuNzQ1LjA5OS0yLjU4OC4yOTIiIGZpbGw9IiNmZjY4NWYiIGZpbGwtcnVsZT0iZXZlbm9kZCIgLz4gPC9zdmc+){.sun} ![](data:image/svg+xml;base64,PHN2ZyBkYXRhLWFzdHJvLWNpZC1pbzY3emozNSB2aWV3Ym94PSIwIDAgMzYgNDEiIGNsYXNzPSJtb29uIj4gPHBhdGggZGF0YS1hc3Ryby1jaWQtaW82N3pqMzUgZmlsbD0iI2ZmNjg1ZiIgZmlsbC1ydWxlPSJldmVub2RkIiBkPSJNMzUuMiAzMy42MDdhMTkuOTIgMTkuOTIgMCAwIDEtMTUuMTE0IDYuOTFDOS4wMzkgNDAuNTE3LjA4OSAzMS41NjYuMDg5IDIwLjUyLjA4OSA5LjY4NyA4LjY5NC44NjggMTkuNDUxLjUzN2ExOS44NzkgMTkuODc5IDAgMCAwLTQuODgyIDEzLjA4OGMwIDExLjA0NyA4Ljk1MSAxOS45OTcgMTkuOTk3IDE5Ljk5Ny4yMTQgMCAuNDI3IDAgLjYzNC0uMDE1IiAvPiA8L3N2Zz4=){.moon}

Click me x5 ↓

# pnpm link

I made my first contribution to Astro the other day. I had to have Astro running locally, and the project uses `pnpm` rather than npm.

It's great because it's a lot faster. I could not figure out for the life of me how to get `pnpm link` to work. So I asked the Astro team on Discord and they helped me. I didn't want this information to stay buried in a Discord chart so I'm making this as a note for myself.

All credit goes to [Bjorn Lu](https://bjornlu.com/){rel="nofollow"} and [Nate Moore](https://natemoo.re/){rel="nofollow"}.

Add this to package json

``` {.astro-code .github-dark data-language="plaintext" tabindex="0" style="background-color:#24292e;color:#e1e4e8; overflow-x: auto;"}
"pnpm": {
  "overrides": {
    "astro": "link:../astro/packages/astro"
  }
},
```

Run this set of commands
TODO

Share ![](data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjAuOWVtIiBhcmlhLWhpZGRlbj0idHJ1ZSIgdmlld2JveD0iMCAwIDUxMiA1MTIiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHBhdGggZD0iTTMyMCA0MDhjMC02LjQyOC0uODQ2LTEyLjY2LTIuNDM0LTE4LjZDMzM4LjIgMzc2LjcgMzUyIDM1My45IDM1MiAzMjhjMC02LjQyOC0uODQ2LTEyLjY2LTIuNDM0LTE4LjZDMzcwLjIgMjk2LjcgMzg0IDI3My45IDM4NCAyNDhjMC0yLjcwNS0uMTQ4LTUuMzczLS40NDEtOEg0NDBjMzkuNyAwIDcyLTMyLjMgNzItNzJzLTMyLjMtNzItNzItNzJIMjQzLjdjLTE2LjItMTkuNDktNDAuNS0zMi02Ny43LTMyaC00OS45Qzk0LjAyIDY0IDY0LjQ3IDgxLjEgNDkgMTA4LjZsLTMxLjM1IDU1LjlDNi4xMDQgMTg1LjEgMCAyMDguNCAwIDIzMS44djEwNy45QzAgNDE3LjEgNjQuNiA0ODAgMTQ0IDQ4MGgxMDRjMzkuNyAwIDcyLTMyLjMgNzItNzJ6bS00MC0xMDRjMTMuMjMgMCAyNCAxMC43OCAyNCAyNHMtMTAuOCAyNC0yNCAyNGgtNDcuOWMtMTMuMiAwLTI0LjEtMTAuOC0yNC4xLTI0czEwLjgtMjQgMjQtMjRoNDh6bTMyLTgwYzEzLjIzIDAgMjQgMTAuNzggMjQgMjRzLTEwLjggMjQtMjQgMjRoLTQ4Yy0zLjAyOSAwLTUuODc1LS43MDEtOC41NDUtMS43M0MyNjAuNyAyNTkuOSAyNjQgMjQ4LjQgMjY0IDIzNnYtMTJoNDh6bTEyOC04MGMxMy4yMyAwIDI0IDEwLjc4IDI0IDI0cy0xMC44IDI0LTI0IDI0SDI2NHYtNDBjMC0yLjY4Ni0uNTU3LTUuMjE3LS43OTMtNy44NC4yOTMuMDQuNDkzLS4xNi43OTMtLjE2aDE3NnpNNDggMzM5LjdWMjMxLjhjMC0xNS4yNSAzLjk4NC0zMC40MSAxMS41Mi00My44OGwzMS4zNC01NS43OEM5Ny44NCAxMTkuNyAxMTEuNCAxMTIgMTI2LjEgMTEySDE3NmMyMi4wNiAwIDQwIDE3Ljk0IDQwIDQwdjg0YzAgMTUuNDQtMTIuNTYgMjgtMjggMjhzLTI4LTEyLjYtMjgtMjh2LTUyYzAtMTMuMi0xMC43LTI0LTI0LTI0cy0yNCAxMC44LTI0IDI0djUyYzAgMzMuMjMgMjEuNTggNjEuMjUgNTEuMzYgNzEuNTRDMTYxLjMgMzE0IDE2MCAzMjAuOSAxNjAgMzI4YzAgNS4wNDEgMS4xNjYgOS44MzYgMi4xNzggMTQuNjZDMTM3LjQgMzU0IDEyMCAzNzguMSAxMjAgNDA4YzAgNy42ODQgMS41NTcgMTQuOTQgMy44MzYgMjEuODdDODAuNTYgNDIwLjkgNDggMzgzLjkgNDggMzM5Ljd6TTE5MiA0MzJjLTEzLjIzIDAtMjQtMTAuNzgtMjQtMjRzMTAuOC0yNCAyNC0yNGg1NmMxMy4yMyAwIDI0IDEwLjc4IDI0IDI0cy0xMC43NyAyNC0yNCAyNGgtNTZ6IiBmaWxsPSJ2YXIoLS1oaWdobGlnaHQpIiAvPjwvc3ZnPg==)

[![](data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjY0IiB3aWR0aD0iNjQiIHZpZXdib3g9IjAgMCA2NCA2NCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiByb2xlPSJpbWciPgogIDxjaXJjbGUgZmlsbD0iI0VGM0Y1NiIgcj0iMzEiIGN5PSIzMiIgY3g9IjMyIj48L2NpcmNsZT4KICA8cGF0aCBmaWxsPSJ3aGl0ZSIgZD0iTTQxLjA4NCAyOS4wNjVsLTcuNTI4IDcuODgyYTIuMTA0IDIuMTA0IDAgMCAxLTEuNTIxLjY2NiAyLjEwNiAyLjEwNiAwIDAgMS0xLjUyMi0uNjY2bC03LjUyOC03Ljg4MmMtLjg3Ni0uOTE0LS45MDItMi40My0uMDY1LTMuMzg0Ljg0LS45NTUgMi4yMjgtLjk4NyAzLjEtLjA3Mmw2LjAxNSA2LjI4NiA2LjAyMi02LjI4NmMuODgtLjkxOCAyLjI2My0uODgzIDMuMTAyLjA3MS44NDEuOTM4LjgyIDIuNDY1LS4wNiAzLjM4M2wtLjAxNS4wMDJ6bTYuNzc3LTEwLjk3NkM0Ny40NjMgMTYuODQgNDYuMzYxIDE2IDQ1LjE0IDE2SDE4LjkwNWMtMS4yIDAtMi4yODkuODItMi43MTYgMi4wNDQtLjEyNS4zNjMtLjE4OS43NDMtLjE4OSAxLjEyNXYxMC41MzlsLjExMiAyLjA5NmMuNDY0IDQuNzY2IDIuNzMgOC45MzMgNi4yNDMgMTEuODM4LjA2LjA1My4xMjUuMTAyLjE5LjE1M2wuMDQuMDMzYzEuODgyIDEuNDk5IDMuOTg2IDIuNTE0IDYuMjU5IDMuMDE0YTE0LjY2MiAxNC42NjIgMCAwIDAgNi4xMy4wNTJjLjExOC0uMDQyLjIzNS0uMDY1LjM1My0uMDg3LjAzIDAgLjA2NS0uMDIyLjA5OC0uMDQyYTE1LjM5NSAxNS4zOTUgMCAwIDAgNi4wMTEtMi45NDVsLjAzOS0uMDQ1LjE4LS4xNTNjMy41MDItMi45MDIgNS43NjUtNy4wNzIgNi4yNDgtMTEuODUyTDQ4IDI5LjY3NHYtMTAuNTJjMC0uMzY2LS4wNDEtLjcyOC0uMTYxLTEuMDhsLjAyMi4wMTV6IiAvPgo8L3N2Zz4=)](https://getpocket.com/save?url=https%3A%2F%2Falvin.codes%2F%2Fsnippets%2Fpnpm-link&title=pnpm+link "Save on Pocket"){.no-icon rel="noopener noreferrer"}[![](data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjY0IiB3aWR0aD0iNjQiIHZpZXdib3g9IjAgMCA2NCA2NCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiByb2xlPSJpbWciPgogIDxjaXJjbGUgZmlsbD0iIzAwYWNlZCIgcj0iMzEiIGN5PSIzMiIgY3g9IjMyIj48L2NpcmNsZT4KICA8cGF0aCBmaWxsPSJ3aGl0ZSIgZD0iTTQ4LDIyLjFjLTEuMiwwLjUtMi40LDAuOS0zLjgsMWMxLjQtMC44LDIuNC0yLjEsMi45LTMuNmMtMS4zLDAuOC0yLjcsMS4zLTQuMiwxLjYgQzQxLjcsMTkuOCw0MCwxOSwzOC4yLDE5Yy0zLjYsMC02LjYsMi45LTYuNiw2LjZjMCwwLjUsMC4xLDEsMC4yLDEuNWMtNS41LTAuMy0xMC4zLTIuOS0xMy41LTYuOWMtMC42LDEtMC45LDIuMS0wLjksMy4zIGMwLDIuMywxLjIsNC4zLDIuOSw1LjVjLTEuMSwwLTIuMS0wLjMtMy0wLjhjMCwwLDAsMC4xLDAsMC4xYzAsMy4yLDIuMyw1LjgsNS4zLDYuNGMtMC42LDAuMS0xLjEsMC4yLTEuNywwLjJjLTAuNCwwLTAuOCwwLTEuMi0wLjEgYzAuOCwyLjYsMy4zLDQuNSw2LjEsNC42Yy0yLjIsMS44LTUuMSwyLjgtOC4yLDIuOGMtMC41LDAtMS4xLDAtMS42LTAuMWMyLjksMS45LDYuNCwyLjksMTAuMSwyLjljMTIuMSwwLDE4LjctMTAsMTguNy0xOC43IGMwLTAuMywwLTAuNiwwLTAuOEM0NiwyNC41LDQ3LjEsMjMuNCw0OCwyMi4xeiIgLz4KPC9zdmc+)](https://twitter.com/share?url=https%3A%2F%2Falvin.codes%2F%2Fsnippets%2Fpnpm-link&text=pnpm+link&via=hola_alvin&hashtags= "Share on twitter"){.no-icon rel="noopener noreferrer"}[![](data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjY0IiB3aWR0aD0iNjQiIHZpZXdib3g9IjAgMCAyMCAyMCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KICA8Y2lyY2xlIHI9IjEwIiBjeT0iMTAiIGN4PSIxMCIgZmlsbD0iI0ZGNDUwMCI+PC9jaXJjbGU+CiAgPHBhdGggZD0iTTE2LjY3LDEwQTEuNDYsMS40NiwwLDAsMCwxNC4yLDlhNy4xMiw3LjEyLDAsMCwwLTMuODUtMS4yM0wxMSw0LjY1LDEzLjE0LDUuMWExLDEsMCwxLDAsLjEzLTAuNjFMMTAuODIsNGEwLjMxLDAuMzEsMCwwLDAtLjM3LjI0TDkuNzEsNy43MWE3LjE0LDcuMTQsMCwwLDAtMy45LDEuMjNBMS40NiwxLjQ2LDAsMSwwLDQuMiwxMS4zM2EyLjg3LDIuODcsMCwwLDAsMCwuNDRjMCwyLjI0LDIuNjEsNC4wNiw1LjgzLDQuMDZzNS44My0xLjgyLDUuODMtNC4wNmEyLjg3LDIuODcsMCwwLDAsMC0uNDRBMS40NiwxLjQ2LDAsMCwwLDE2LjY3LDEwWm0tMTAsMWExLDEsMCwxLDEsMSwxQTEsMSwwLDAsMSw2LjY3LDExWm01LjgxLDIuNzVhMy44NCwzLjg0LDAsMCwxLTIuNDcuNzcsMy44NCwzLjg0LDAsMCwxLTIuNDctLjc3LDAuMjcsMC4yNywwLDAsMSwuMzgtMC4zOEEzLjI3LDMuMjcsMCwwLDAsMTAsMTRhMy4yOCwzLjI4LDAsMCwwLDIuMDktLjYxQTAuMjcsMC4yNywwLDEsMSwxMi40OCwxMy43OVptLTAuMTgtMS43MWExLDEsMCwxLDEsMS0xQTEsMSwwLDAsMSwxMi4yOSwxMi4wOFoiIGZpbGw9IiNGRkYiPgogIDwvcGF0aD4KPC9zdmc+)](https://www.reddit.com/submit?url=https%3A%2F%2Falvin.codes%2F%2Fsnippets%2Fpnpm-link&title=pnpm+link "Share on Reddit"){.no-icon rel="noopener noreferrer"}[![](data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjY0IiB3aWR0aD0iNjQiIHZpZXdib3g9IjAgMCA2NCA2NCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiByb2xlPSJpbWciPgogIDxjaXJjbGUgZmlsbD0iIzAwN2ZiMSIgcj0iMzEiIGN5PSIzMiIgY3g9IjMyIj48L2NpcmNsZT4KICA8cGF0aCBmaWxsPSJ3aGl0ZSIgZD0iTTIwLjQsNDRoNS40VjI2LjZoLTUuNFY0NHogTTIzLjEsMThjLTEuNywwLTMuMSwxLjQtMy4xLDMuMWMwLDEuNywxLjQsMy4xLDMuMSwzLjEgYzEuNywwLDMuMS0xLjQsMy4xLTMuMUMyNi4yLDE5LjQsMjQuOCwxOCwyMy4xLDE4eiBNMzkuNSwyNi4yYy0yLjYsMC00LjQsMS40LTUuMSwyLjhoLTAuMXYtMi40aC01LjJWNDRoNS40di04LjYgYzAtMi4zLDAuNC00LjUsMy4yLTQuNWMyLjgsMCwyLjgsMi42LDIuOCw0LjZWNDRINDZ2LTkuNUM0NiwyOS44LDQ1LDI2LjIsMzkuNSwyNi4yeiIgLz4KPC9zdmc+)](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Falvin.codes%2F%2Fsnippets%2Fpnpm-link&title=pnpm+link "Share on LinkedIn"){.no-icon rel="noopener noreferrer"}

### Email List

If you had an RSS reader, this wouldn't be here. But let's be real.\
People do email lists now.

First Name

Email

Don't fill this out if you\'re human:

Join

**Contact**\
[Email List](https://alv.sh/email-list "Let's be internet friends"){.no-icon}\
<hello@alvin.codes>\
© 2016--2024

**More**\
-- [/now](/now "Visit my now page")\
-- [/uses](/uses "Visit my uses page")

[![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdib3g9IjAgMCAyNCAyNCIgcm9sZT0iaW1nIj48cGF0aCBkPSJNMTIgLjI5N2MtNi42MyAwLTEyIDUuMzczLTEyIDEyIDAgNS4zMDMgMy40MzggOS44IDguMjA1IDExLjM4NS42LjExMy44Mi0uMjU4LjgyLS41NzcgMC0uMjg1LS4wMS0xLjA0LS4wMTUtMi4wNC0zLjMzOC43MjQtNC4wNDItMS42MS00LjA0Mi0xLjYxQzQuNDIyIDE4LjA3IDMuNjMzIDE3LjcgMy42MzMgMTcuN2MtMS4wODctLjc0NC4wODQtLjcyOS4wODQtLjcyOSAxLjIwNS4wODQgMS44MzggMS4yMzYgMS44MzggMS4yMzYgMS4wNyAxLjgzNSAyLjgwOSAxLjMwNSAzLjQ5NS45OTguMTA4LS43NzYuNDE3LTEuMzA1Ljc2LTEuNjA1LTIuNjY1LS4zLTUuNDY2LTEuMzMyLTUuNDY2LTUuOTMgMC0xLjMxLjQ2NS0yLjM4IDEuMjM1LTMuMjItLjEzNS0uMzAzLS41NC0xLjUyMy4xMDUtMy4xNzYgMCAwIDEuMDA1LS4zMjIgMy4zIDEuMjMuOTYtLjI2NyAxLjk4LS4zOTkgMy0uNDA1IDEuMDIuMDA2IDIuMDQuMTM4IDMgLjQwNSAyLjI4LTEuNTUyIDMuMjg1LTEuMjMgMy4yODUtMS4yMy42NDUgMS42NTMuMjQgMi44NzMuMTIgMy4xNzYuNzY1Ljg0IDEuMjMgMS45MSAxLjIzIDMuMjIgMCA0LjYxLTIuODA1IDUuNjI1LTUuNDc1IDUuOTIuNDIuMzYuODEgMS4wOTYuODEgMi4yMiAwIDEuNjA2LS4wMTUgMi44OTYtLjAxNSAzLjI4NiAwIC4zMTUuMjEuNjkuODI1LjU3QzIwLjU2NSAyMi4wOTIgMjQgMTcuNTkyIDI0IDEyLjI5N2MwLTYuNjI3LTUuMzczLTEyLTEyLTEyIiAvPjwvc3ZnPg==)](https://github.com/alvinometric "Stalk on GitHub"){.no-icon rel="me nofollow"}[![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdib3g9IjAgMCAyNCAyNCIgcm9sZT0iaW1nIj48cGF0aCBkPSJNMjMuOTU0IDQuNTY5Yy0uODg1LjM4OS0xLjgzLjY1NC0yLjgyNS43NzUgMS4wMTQtLjYxMSAxLjc5NC0xLjU3NCAyLjE2My0yLjcyMy0uOTUxLjU1NS0yLjAwNS45NTktMy4xMjcgMS4xODQtLjg5Ni0uOTU5LTIuMTczLTEuNTU5LTMuNTkxLTEuNTU5LTIuNzE3IDAtNC45MiAyLjIwMy00LjkyIDQuOTE3IDAgLjM5LjA0NS43NjUuMTI3IDEuMTI0QzcuNjkxIDguMDk0IDQuMDY2IDYuMTMgMS42NCAzLjE2MWMtLjQyNy43MjItLjY2NiAxLjU2MS0uNjY2IDIuNDc1IDAgMS43MS44NyAzLjIxMyAyLjE4OCA0LjA5Ni0uODA3LS4wMjYtMS41NjYtLjI0OC0yLjIyOC0uNjE2di4wNjFjMCAyLjM4NSAxLjY5MyA0LjM3NCAzLjk0NiA0LjgyNy0uNDEzLjExMS0uODQ5LjE3MS0xLjI5Ni4xNzEtLjMxNCAwLS42MTUtLjAzLS45MTYtLjA4Ni42MzEgMS45NTMgMi40NDUgMy4zNzcgNC42MDQgMy40MTctMS42OCAxLjMxOS0zLjgwOSAyLjEwNS02LjEwMiAyLjEwNS0uMzkgMC0uNzc5LS4wMjMtMS4xNy0uMDY3IDIuMTg5IDEuMzk0IDQuNzY4IDIuMjA5IDcuNTU3IDIuMjA5IDkuMDU0IDAgMTMuOTk5LTcuNDk2IDEzLjk5OS0xMy45ODYgMC0uMjA5IDAtLjQyLS4wMTUtLjYzLjk2MS0uNjg5IDEuOC0xLjU2IDIuNDYtMi41NDhsLS4wNDctLjAyeiIgLz48L3N2Zz4=)](https://twitter.com/hola_alvin "Plz follow on Twitter"){.no-icon rel="me nofollow"}[![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdib3g9IjAgMCAyNCAyNCIgcm9sZT0iaW1nIj48dGl0bGU+SW5zdGFncmFtPC90aXRsZT48cGF0aCBkPSJNNy4wMzAxLjA4NGMtMS4yNzY4LjA2MDItMi4xNDg3LjI2NC0yLjkxMS41NjM0LS43ODg4LjMwNzUtMS40NTc1LjcyLTIuMTIyOCAxLjM4NzctLjY2NTIuNjY3Ny0xLjA3NSAxLjMzNjgtMS4zODAyIDIuMTI3LS4yOTU0Ljc2MzgtLjQ5NTYgMS42MzY1LS41NTIgMi45MTQtLjA1NjQgMS4yNzc1LS4wNjg5IDEuNjg4Mi0uMDYyNiA0Ljk0Ny4wMDYyIDMuMjU4Ni4wMjA2IDMuNjY3MS4wODI1IDQuOTQ3My4wNjEgMS4yNzY1LjI2NCAyLjE0ODIuNTYzNSAyLjkxMDcuMzA4Ljc4ODkuNzIgMS40NTczIDEuMzg4IDIuMTIyOC42Njc5LjY2NTUgMS4zMzY1IDEuMDc0MyAyLjEyODUgMS4zOC43NjMyLjI5NSAxLjYzNjEuNDk2MSAyLjkxMzQuNTUyIDEuMjc3My4wNTYgMS42ODg0LjA2OSA0Ljk0NjIuMDYyNyAzLjI1NzgtLjAwNjIgMy42NjgtLjAyMDcgNC45NDc4LS4wODE0IDEuMjgtLjA2MDcgMi4xNDctLjI2NTIgMi45MDk4LS41NjMzLjc4ODktLjMwODYgMS40NTc4LS43MiAyLjEyMjgtMS4zODgxLjY2NS0uNjY4MiAxLjA3NDUtMS4zMzc4IDEuMzc5NS0yLjEyODQuMjk1Ny0uNzYzMi40OTY2LTEuNjM2LjU1Mi0yLjkxMjQuMDU2LTEuMjgwOS4wNjkyLTEuNjg5OC4wNjMtNC45NDgtLjAwNjMtMy4yNTgzLS4wMjEtMy42NjY4LS4wODE3LTQuOTQ2NS0uMDYwNy0xLjI3OTctLjI2NC0yLjE0ODctLjU2MzMtMi45MTE3LS4zMDg0LS43ODg5LS43Mi0xLjQ1NjgtMS4zODc2LTIuMTIyOEMyMS4yOTgyIDEuMzMgMjAuNjI4LjkyMDggMTkuODM3OC42MTY1IDE5LjA3NC4zMjEgMTguMjAxNy4xMTk3IDE2LjkyNDQuMDY0NSAxNS42NDcxLjAwOTMgMTUuMjM2LS4wMDUgMTEuOTc3LjAwMTQgOC43MTguMDA3NiA4LjMxLjAyMTUgNy4wMzAxLjA4MzltLjE0MDIgMjEuNjkzMmMtMS4xNy0uMDUwOS0xLjgwNTMtLjI0NTMtMi4yMjg3LS40MDgtLjU2MDYtLjIxNi0uOTYtLjQ3NzEtMS4zODE5LS44OTUtLjQyMi0uNDE3OC0uNjgxMS0uODE4Ni0uOS0xLjM3OC0uMTY0NC0uNDIzNC0uMzYyNC0xLjA1OC0uNDE3MS0yLjIyOC0uMDU5NS0xLjI2NDUtLjA3Mi0xLjY0NDItLjA3OS00Ljg0OC0uMDA3LTMuMjAzNy4wMDUzLTMuNTgzLjA2MDctNC44NDguMDUtMS4xNjkuMjQ1Ni0xLjgwNS40MDgtMi4yMjgyLjIxNi0uNTYxMy40NzYyLS45Ni44OTUtMS4zODE2LjQxODgtLjQyMTcuODE4NC0uNjgxNCAxLjM3ODMtLjkwMDMuNDIzLS4xNjUxIDEuMDU3NS0uMzYxNCAyLjIyNy0uNDE3MSAxLjI2NTUtLjA2IDEuNjQ0Ny0uMDcyIDQuODQ4LS4wNzkgMy4yMDMzLS4wMDcgMy41ODM1LjAwNSA0Ljg0OTUuMDYwOCAxLjE2OS4wNTA4IDEuODA1My4yNDQ1IDIuMjI4LjQwOC41NjA4LjIxNi45Ni40NzU0IDEuMzgxNi44OTUuNDIxNy40MTk0LjY4MTYuODE3Ni45MDA1IDEuMzc4Ny4xNjUzLjQyMTcuMzYxNyAxLjA1Ni40MTY5IDIuMjI2My4wNjAyIDEuMjY1NS4wNzM5IDEuNjQ1LjA3OTYgNC44NDguMDA1OCAzLjIwMy0uMDA1NSAzLjU4MzQtLjA2MSA0Ljg0OC0uMDUxIDEuMTctLjI0NSAxLjgwNTUtLjQwOCAyLjIyOTQtLjIxNi41NjA0LS40NzYzLjk2LS44OTU0IDEuMzgxNC0uNDE5LjQyMTUtLjgxODEuNjgxMS0xLjM3ODMuOS0uNDIyNC4xNjQ5LTEuMDU3Ny4zNjE3LTIuMjI2Mi40MTc0LTEuMjY1Ni4wNTk1LTEuNjQ0OC4wNzItNC44NDkzLjA3OS0zLjIwNDUuMDA3LTMuNTgyNS0uMDA2LTQuODQ4LS4wNjA4TTE2Ljk1MyA1LjU4NjRBMS40NCAxLjQ0IDAgMSAwIDE4LjM5IDQuMTQ0YTEuNDQgMS40NCAwIDAgMC0xLjQzNyAxLjQ0MjRNNS44Mzg1IDEyLjAxMmMuMDA2NyAzLjQwMzIgMi43NzA2IDYuMTU1NyA2LjE3MyA2LjE0OTMgMy40MDI2LS4wMDY1IDYuMTU3LTIuNzcwMSA2LjE1MDYtNi4xNzMzLS4wMDY1LTMuNDAzMi0yLjc3MS02LjE1NjUtNi4xNzQtNi4xNDk4LTMuNDAzLjAwNjctNi4xNTYgMi43NzEtNi4xNDk2IDYuMTczOE04IDEyLjAwNzdhNCA0IDAgMSAxIDQuMDA4IDMuOTkyMUEzLjk5OTYgMy45OTk2IDAgMCAxIDggMTIuMDA3NyIgLz48L3N2Zz4=)](https://instagram.com/hola_alvin "Instagram, sadly"){.no-icon rel="me nofollow"}[![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdib3g9IjAgMCAyNCAyNCIgcm9sZT0iaW1nIj48dGl0bGU+UlNTIGljb248L3RpdGxlPjxwYXRoIGQ9Ik0xOS4xOTkgMjRDMTkuMTk5IDEzLjQ2NyAxMC41MzMgNC44IDAgNC44VjBjMTMuMTY1IDAgMjQgMTAuODM1IDI0IDI0aC00LjgwMXpNMy4yOTEgMTcuNDE1YzEuODE0IDAgMy4yOTMgMS40NzkgMy4yOTMgMy4yOTUgMCAxLjgxMy0xLjQ4NSAzLjI5LTMuMzAxIDMuMjlDMS40NyAyNCAwIDIyLjUyNiAwIDIwLjcxczEuNDc1LTMuMjk0IDMuMjkxLTMuMjk1ek0xNS45MDkgMjRoLTQuNjY1YzAtNi4xNjktNS4wNzUtMTEuMjQ1LTExLjI0NC0xMS4yNDVWOC4wOWM4LjcyNyAwIDE1LjkwOSA3LjE4NCAxNS45MDkgMTUuOTF6IiAvPjwvc3ZnPg==)](/rss.xml "Yes, there's an RSS Feed"){.no-icon rel="me"} [![](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdib3g9IjAgMCA1NjYgNTAwIiBjbGFzcz0iYmx1ZXNreS1mbHV0dGVyIiBpZD0iZmx1dHRlcmJ5Ij4gPGRlZnM+IDxwYXRoIGQ9Ik0gMTIzLjI0NCAzNS4wMDggQyAxODguMjQ4IDgzLjgwOSAyODMuODM2IDE3Ni44NzkgMjgzLjgzNiAyMzUuODU3IEMgMjgzLjgzNiAzMTYuODk5IDI4My44NzkgMjM1Ljg0NSAyODMuODM2IDM3Ni4wMzggQyAyODMuODg5IDM3NS45OTUgMjgyLjY3IDM3Ni41NDQgMjgwLjIxMiAzODMuNzU4IEMgMjY2LjgwNiA0MjMuMTExIDIxNC40ODcgNTc2LjY4NSA5NC44NDEgNDUzLjkxMyBDIDMxLjg0MyAzODkuMjY5IDYxLjAxMyAzMjQuNjI1IDE3NS42ODIgMzA1LjEwOCBDIDExMC4wOCAzMTYuMjc0IDM2LjMzMiAyOTcuODI3IDE2LjA5MyAyMjUuNTA0IEMgMTAuMjcxIDIwNC42OTkgMC4zNDMgNzYuNTYgMC4zNDMgNTkuMjQ2IEMgMC4zNDMgLTI3LjQ1MSA3Ni4zNDIgLTAuMjA2IDEyMy4yNDQgMzUuMDA4IFoiIGZpbGw9ImN1cnJlbnRDb2xvciIgaWQ9IndpbmciIC8+IDwvZGVmcz4gICA8L3N2Zz4=){#flutterby .bluesky-flutter}](https://bsky.app/profile/alvin.codes){.no-icon ._bluesky-flutter_bbs6n_59}