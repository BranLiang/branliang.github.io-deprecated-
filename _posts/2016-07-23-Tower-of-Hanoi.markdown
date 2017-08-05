---
layout:     post
title:      "Tower of Hanoi with Ruby"
---

## Intro

The Tower of Hanoi is a mathematical game or puzzle. It consists of three rods, and a number of disks of different sizes which can slide onto any rod. The puzzle starts with the disks in a neat stack in ascending order of size on one rod, the smallest at the top, thus making a conical shape.
The objective of the puzzle is to move the entire stack to another rod, obeying the following simple rules:

 * Only one disk can be moved at a time.

 * Each move consists of taking the upper disk from one of the stacks and placing it on top of another stack i.e. a disk can only be moved if it is the uppermost disk on a stack.

 * No disk may be placed on top of a smaller disk.

With three disks, the puzzle can be solved in seven moves. The minimum number of moves required to solve a Tower of Hanoi puzzle is 2n - 1, where n is the number of disks.

> This comes from the [Wikipedia](https://en.wikipedia.org/wiki/Tower_of_Hanoi) of course.

## Build it with Ruby and love


* [Update OOP version 07-25-2016](https://github.com/BranLiang/assignment_oop_warmups_1/blob/master/ref_toh.rb)
* [link to my solution](https://github.com/BranLiang/assignment_toh)
* And really do play it following the instruction. It is quiet fun XD.

![toh2](/img/posts/toh1.png)
<small class="img-hint">Tower of Hanoi game play<small>

So what I am going to do is to build it with ruby? But where the hell should I start?
 * First of all, the disk and column should be described in a more measureable form. Obviously, number is the most measureable stuff. So the disks can be named with different numbers because they are with different sizes. And imagining that the column can hold so many different number. Obviously, the column is either an array or a hash. I choose array, what do you think? And that's it, the initialization is done.

```ruby
def initialize num_disks
  @num_disks  = num_disks
  @ini_arrange = Array(1..num_disks)
  @col_1 = @ini_arrange
  @col_2 = []
  @col_3 = []
end
```

I think when you get this idea, then everything would be clear. You need to be able to move the first number of an array. You also need to validate each move. Finally you also need to be able to tell if the game is over or not. But, the fun stuff is how to visulize it? Emmm, I thinked for a while and made few drawings, Bing!

```
   =|=
  ==|==
 ===|===
====|====
```

After few minutes, I think I got the solution. Just puts the disk with "=" and things done. One tricky part is that you need to fill all the column with '0' and all column should be with the same size, otherwise, I could not print it out properly. So the final solution for me it as follows.

```ruby
  def array_fill arr
    result = []
    @num_disks.times do |i|
      if arr[i] != nil
        result << arr[i]
      else
        result << 0
      end
    end
    result
  end

  def display_one_line n
    move_distance = @num_disks + 2
    unless n.nil?
      ("=" * n).rjust(move_distance) + "|" + ("=" * n).ljust(move_distance)
    else
      " ".rjust(move_distance) + "|" + " ".ljust(move_distance)
    end
  end #I was stupid here, actually it can just be populated from the middle.

  def display_all
    @num_disks.times do |i|
      print display_one_line array_fill(col_1).sort[i]
      print display_one_line array_fill(col_2).sort[i]
      print display_one_line array_fill(col_3).sort[i]
      puts
    end
  end
```

## Done

It is just a small fun code game. Be sure to leave a comment if you like it or would like to help modify it:)
And again try to play the game build with love.
[link](https://github.com/BranLiang/assignment_toh/blob/master/towers.rb)
