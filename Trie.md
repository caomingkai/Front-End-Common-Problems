---
title: Trie
comments: true
date: 2018-08-31 10:07:43
type: categories
categories: Algorithm
tags: Trie
---



## what

Trie, is actually a more general tree, also **prefix tree**.

Used for ***massive*** words process, dictionary representation, and lexicographic sorting, word frequency calc.



## How

#### I. TrieNode

TrieNode has mainly two parts:

1. `children`, to link next possible characters appear in a words. If only considering 'A-Z', there is 26 children at most
2. info used to record current TrieNode
   + `isWord`, denotes previous character is a word or not.
   + `cnt`, denotes number of words consist of current prefix

***Note:*** this representation is a little different from binary tree, in which info data are stored for current Node. However, in Trie, info data are stored for previous Node, as follows:

```java
For dictionary: { "a", "bd", "c" }, the Trie would be represented as following:

root TrieNode(0xefe4)
- cnt: 0
- isWord: false
- children:{'a': TrieNode , 'b': TrieNode, 'c': TrieNode, ... }
            /				      |				          \
            TrieNode(0xffe3)	TrieNode(0xffe4)   			TrieNode(0xffe5)
            - cnt: 1			- cnt: 0					- cnt: 1
            - isWord: true		- isWord: false				- isWord: false
            - children: null	- children: {'d':TrieNode}  - children: null
                                    |
                                   ...
```

## Implementation

- interfaces

  - build (constructor)
  - isWord
  - countWord
  - insert
  - delete
  - hasPrefix
  - getPrefixNode

- Time Complexity: [ `n` words, `K` is number of characters of the longest word ]

  - build: O(nK)
  - isWord: O(K)
  - countWord: O(K)
  - insert: O(K)
  - delete: O(K)

- Space Complexity: [`N` is the number of all characters appears in the dictionary ]

  - too much space: O(N) +O(N^2) +O(N^3) +O(N^4) + O(N^K)
  - good news is: when it is a really dictionary, all the space will be filled up, so not so much waste

- `delete` might be a little complex. The to-be-deleted word possibly :

  ```java
  when need delete a node, what should really to be done????
      - delete the key from HashMap<Character, Node>
      - reset to null when using Node[]
  So, always ask two questions:
  	- the node to be deleted, is part of other long word [by checking 'children']?
      - the node itself is the end of a word? [by checking 'isWord' & 'cnt']?
  	
  ```



  - `null `, empty string, not exists in the Trie
  - contains other short words: "abc", "ab"
    - **recursively from bottom to top** delete node, till the longest prefix
  - has same words as itself: "abc", "abc"
    - don't delete node, only update `cnt`
  - being a prefix of other long words: "abc", "abcde"
    - don't delete node, only update `isWord`
  - share prefix with other words: "abc", "abd"
    - **recursively from bottom to top** delete node, till the longest prefix
  - only itself in the dictionary, not affect others "abc"
    - delte all nodes


```java

import java.util.*;

class Node {
	public int cnt;
	public boolean isWord;
	public Map<Character, Node> children;

	public Node(){
		this.cnt = 0;
		this.isWord = false;
		this.children = new HashMap<>();
	}
}


class Trie{
	Node root;
	public Trie( String[] strArr ){
		this.root = new Node();
		for( String str : strArr ){
			insert( str );
		}
	}

	public boolean hasPrefix(String str ){
		if( str == null || str.length() == 0 ) return true;
		int len = str.length();
		Node cur = this.root;
		for( int i = 0; i < len; i++ ){
			char ch = str.charAt(i);
			if( !cur.children.containsKey(ch) ) return false;
			cur = cur.children.get(ch);
		}
		return true;
	}

	public Node getPrefixNode (String str){
		if( !hasPrefix(str) ) return null;
		int len = str.length();
		Node cur = this.root;
		for( int i = 0; i < len; i++ ){
			char ch = str.charAt(i);
			cur = cur.children.get(ch);
		}
		return cur;
	}

	public void insert( String str ){
		if( str == null || str.length() == 0 ) return;
		int len = str.length();
		Node cur = this.root;
		for( int i = 0; i < len; i++ ){
			char ch = str.charAt(i);
			if( !cur.children.containsKey(ch) ){
				cur.children.put( ch, new Node() );
			}
			cur = cur.children.get(ch);
		}
		cur.cnt++;
		cur.isWord = true;
	}

	public void delete (String str ){
		if( str == null || str.length() == 0 ) return;
		if( !hasPrefix(str) ) return;
		Node cur = getPrefixNode(str);
		cur.cnt--; 
		if(cur.cnt < 0) cur.cnt = 0;
		if(cur.cnt > 0) return; 	// at this position, there still word exist
		else{						// at this position, there no word exist
			if( cur.children.size() > 0 ) {	// but can be prefix of other words
				cur.isWord = false;
				return;
			}else{							// not a word, nor a prefix of others
				deleteNode(str);
			}
		}
		
	}
	public void deleteNode( String str ){
		int len = str.length();
		if( len == 0 ) return;

		String prevStr = str.substring(0, len-1);
		Node prevNode = getPrefixNode(prevStr);
		prevNode.children.remove(str.charAt(len-1));
		if( prevNode.children.size()==0 && prevNode.cnt==0 ) delete(prevStr);
		return;
	}
	

	public boolean isWord( String str ){
		if( !hasPrefix(str) ) return false;
		return getPrefixNode(str).isWord;
	}	

	public int countWord( String str ){
		if( !hasPrefix(str) ) return 0;
		return getPrefixNode(str).cnt;
	}

	
}

class TrieTree{
	public static void main( String[] args ){
		String[] dict = { "a", "ab", "ab", "abc", "abc", "abc", "abcd", "ac", "acd", "ad", "2,24",  "2,2,4" };
		Trie trieTree = new Trie(dict);
		
		// String isWordTest1 = "ab";
		// String isWordTest2 = "";
		// String isWordTest3 = "a";
		// String isWordTest4 = "abcd";
		// String isWordTest5 = "abcde";

		// String countTest = "abc";
		// String insertTest = "abc";
		

		// System.out.println( trieTree.isWord(isWordTest1));
		// System.out.println( trieTree.isWord(isWordTest2));
		// System.out.println( trieTree.isWord(isWordTest3));
		// System.out.println( trieTree.isWord(isWordTest4));
		// System.out.println( trieTree.isWord(isWordTest5));


		// System.out.println( trieTree.countWord(countTest));
		// trieTree.insert(insertTest);
		// System.out.println( trieTree.countWord(countTest));
		// trieTree.insert(insertTest);
		// System.out.println( trieTree.countWord(countTest));


		// String hasPrefixTest1 = "abcd";
		// String hasPrefixTest2 = "2,2";
		// String hasPrefixTest3 = "abcdd";
		// System.out.println( trieTree.hasPrefix(hasPrefixTest1));
		// System.out.println( trieTree.hasPrefix(hasPrefixTest2));
		// System.out.println( trieTree.hasPrefix(hasPrefixTest3));


		String deleteTest1 = "abcd";
		String deleteTest2 = "abc";
		String deleteTest3 = "ab";
		trieTree.delete(deleteTest1);
		System.out.println( trieTree.countWord(deleteTest3)); // 2
		System.out.println( trieTree.countWord(deleteTest2)); // 3
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();

		trieTree.delete(deleteTest1);
		System.out.println( trieTree.countWord(deleteTest3)); // 2
		System.out.println( trieTree.countWord(deleteTest2)); // 3
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();

		trieTree.delete(deleteTest2);
		System.out.println( trieTree.countWord(deleteTest3)); // 2
		System.out.println( trieTree.countWord(deleteTest2)); // 2
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
		trieTree.delete(deleteTest2);
		System.out.println( trieTree.countWord(deleteTest3)); // 2
		System.out.println( trieTree.countWord(deleteTest2)); // 1
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
		
		trieTree.delete(deleteTest2);
		System.out.println( trieTree.countWord(deleteTest3)); // 2
		System.out.println( trieTree.countWord(deleteTest2)); // 0
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
		

		trieTree.delete(deleteTest2);
		System.out.println( trieTree.countWord(deleteTest3)); // 2
		System.out.println( trieTree.countWord(deleteTest2)); // 0
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
		

		trieTree.delete(deleteTest3);
		System.out.println( trieTree.countWord(deleteTest3)); // 1
		System.out.println( trieTree.countWord(deleteTest2)); // 0
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
		

		trieTree.delete(deleteTest3);
		System.out.println( trieTree.countWord(deleteTest3)); // 0
		System.out.println( trieTree.countWord(deleteTest2)); // 0
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
		

		trieTree.delete(deleteTest1);
		System.out.println( trieTree.countWord(deleteTest3)); // 0
		System.out.println( trieTree.countWord(deleteTest2)); // 0
		System.out.println( trieTree.countWord(deleteTest1)); // 0
		System.out.println();
		
	}
}
```



