---
title: 更改 alfred 默认 shell 为 iTerm
tags:
  - 效率
  - alfred
  - shell
  - iTerm
keywords:
  - 效率
  - alfred
  - shell
  - iTerm
categories:
  - 工具
abbrlink: 70f00860
date: 2019-05-02 17:39:49
---

## 更改 alfred 默认 shell 为 iTerm2 
上码记录
```
on alfred_script(q)
		run script "
			on run {q}
				tell application \":Applications:iTerm.app\"
					activate
					try
						select first window
					on error
						create window with default profile
						select first window
					end try
					tell the first window
						tell current session to write text q
					end tell
				end tell
			end run
		" with parameters {q}
end alfred_script
```
