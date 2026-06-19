---
author: Kit
categories:
- Uncategorized
tags:
- Golang
date: 2013-04-14T10:28:09Z
guid: http://kitmenke.com/blog/?p=575
id: 575
title: Using slices in Golang
---

I started helping my brother with some GO projects. One really interesting concept in Go are slices... which make working with arrays easier.

At anytime you can &#8220;chop&#8221; out interesting portions of a slice to use using [start:finish]. 

There is the special append keyword to add things onto the end of the slice dynamically (as opposed to using the make keyword to declare it up front). 

And there is a range keyword to mimic how foreach works in a higher level language.

```go
package main
  
 import (
     "fmt"
 )
 
 func PrintSliceRange(slice []string) {
     fmt.Printf("Slice length = %d\r\n", len(slice))
     for _, s := range slice {
         fmt.Printf("%s\r\n", s)
     }
 }
 
 func PrintSlice(slice []string) {
     fmt.Printf("Slice length = %d\r\n", len(slice))
     for i := ; i < len(slice); i++ {
         fmt.Printf("[%d] := %s\r\n", i, slice[i])
     }
 }
 
 func main() {
     var slice []string
     slice = append(slice, "alpha")
     slice = append(slice, "bravo")
     slice = append(slice, "charlie")
     slice = append(slice, "delta")
     slice = append(slice, "echo")
     fmt.Println(slice)
     /* prints out:
     [alpha bravo charlie delta echo]
     */
     fmt.Println(slice[1:3])
     /* prints out:
     [bravo charlie]
     */
     fmt.Println(slice[3:])
     /* prints out:
     [delta echo]
     */
     PrintSlice(slice)
     /* prints out:
     Slice length = 5
     [0] := alpha
     [1] := bravo
     [2] := charlie
     [3] := delta
     [4] := echo
     */
     PrintSliceRange(slice)
     /* prints out:
     Slice length = 5
     alpha
     bravo
     charlie
     delta
     echo
     */
 }
```

If you want to learn more, definitely check out the [Golang Book's chapter on slices](http://www.golang-book.com/6#section2).