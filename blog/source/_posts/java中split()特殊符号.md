---
title: java中split()特殊符号"." "|" "*" "\" "]"
date: 2016-08-29 17:59:44
tags: java
---

>工作中突然发现split不能正常使用了，查看了源码才发现是特殊字符，对于这种情况可以以下方法解决：

1. 关于点的问题是用string.split("[.]") 解决。
2. 关于竖线的问题用 string.split("\\|")解决。
3. 关于星号的问题用 string.split("\\*")解决。
4. 关于斜线的问题用 sring.split("\\\\")解决。
5. 关于中括号的问题用 sring.split("[\\[\\](file://[//)]")解决。

研究下java 源码可以发现遇到这些特殊字符会抛出异常

<!--more-->


	public final class Pattern
	    implements java.io.Serializable
	{
	    private Node sequence(Node end) {
	        Node head = null;
	        Node tail = null;
	        Node node = null;
	    LOOP:
	        for (;;) {
	            int ch = peek();
	            switch (ch) {
	            case '(':
	                // Because group handles its own closure,
	                // we need to treat it differently
	                node = group0();
	                // Check for comment or flag group
	                if (node == null)
	                    continue;
	                if (head == null)
	                    head = node;
	                else
	                    tail.next = node;
	                // Double return: Tail was returned in root
	                tail = root;
	                continue;
	            case '[':
	                node = clazz(true);
	                break;
	            case '\\':
	                ch = nextEscaped();
	                if (ch == 'p' || ch == 'P') {
	                    boolean oneLetter = true;
			    boolean comp = (ch == 'P');
	                    ch = next(); // Consume { if present
	                    if (ch != '{') {
	                        unread();
	                    } else {
	                        oneLetter = false;
	                    }
			    node = family(oneLetter).maybeComplement(comp);
	                } else {
	                    unread();
	                    node = atom();
	                }
	                break;
	            case '^':
	                next();
	                if (has(MULTILINE)) {
	                    if (has(UNIX_LINES))
	                        node = new UnixCaret();
	                    else
	                        node = new Caret();
	                } else {
	                    node = new Begin();
	                }
	                break;
	            case '$':
	                next();
	                if (has(UNIX_LINES))
	                    node = new UnixDollar(has(MULTILINE));
	                else
	                    node = new Dollar(has(MULTILINE));
	                break;
	            case '.':
	                next();
	                if (has(DOTALL)) {
	                    node = new All();
	                } else {
	                    if (has(UNIX_LINES))
	                        node = new UnixDot();
	                    else {
	                        node = new Dot();
	                    }
	                }
	                break;
	            case '|':
	            case ')':
	                break LOOP;
	            case ']': // Now interpreting dangling ] and } as literals
	            case '}':
	                node = atom();
	                break;
	            case '?':
	            case '*':
	            case '+':
	                next();
	                throw error("Dangling meta character '" + ((char)ch) + "'");
	            case 0:
	                if (cursor >= patternLength) {
	                    break LOOP;
	                }
	                // Fall through
	            default:
	                node = atom();
	                break;
	            }
	
	            node = closure(node);
	
	            if (head == null) {
	                head = tail = node;
	            } else {
	                tail.next = node;
	                tail = node;
	            }
	        }
	        if (head == null) {
	            return end;
	        }
	        tail.next = end;
	        root = tail;      //double return
	        return head;
	    }
	}
