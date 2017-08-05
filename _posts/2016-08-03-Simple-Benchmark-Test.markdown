---
layout:     post
title:      "Simple big data benchmark test"
---

## Intro
A simple benchmark test was carried on here between Class, Struct and Hashes. The idea is very simple, first generate a set of data including lots of :x and :y coordinate. Then, I will reverse the data to get another different coordinates with the same length. The test will calculate the distance for the two data sets on the same position. The only difference is there create method. As is shown in the title that one is using traditional class. One is using the simple struct. The last group is using hash.
[link to the git repository](https://github.com/BranLiang/Simple-big-data-benchmark-test)

## Test Environment
 * System: Linux 16.04 with Gnome
 * CPU: Inter Intel® Core™ i5-5200U CPU @ 2.20GHz × 4
 * Datasets: 10, 100, 1000, 100k

```ruby
require 'yaml'
class DataGenerator
  def generate
    data = []
    100_000.times do
      data << [rand(1000), rand(1000)]
    end
    data = data.to_yaml
    File.open('100k.txt', 'w') do |file|
      file.write data
    end
  end
end
```


 * 3 Different coordinate create methods

```ruby
 class Coordinate
  attr_accessor :x, :y
  def initialize x, y
    @x = x
    @y = y
  end
 end

Coordinate = Struct.new :x, :y

coordinate = { :x, :y }
```

* the simple test function which calculates the distance for each related points

```ruby
  def test
    test_data1 = prepare_work
    test_data2 = test_data1.reverse
    test_time = Benchmark.measure do
      test_data1.each_index do |index|
        distance test_data1[index], test_data2[index]
      end
    end
    puts test_time
  end
```

## Test result
The test results is the elapsed real time as is describe in the following picture. And also the units here is microseconds which is 0.000_001 second.
![benchmark_ruby_result](/img/posts/benchmark-breakdown.jpg)

The results list in the table form.

|        | 10 | 100 | 1000 | 100k  |
|--------|----|-----|------|-------|
| Class  | 14 | 34  | 282  | 27717 |
| Struct | 16 | 34  | 297  | 27861 |
| Hash   | 10 | 35  | 320  | 33763 |

## Conclusion

For this simple benchmark test. Obviously, the Class and Struct beat the hash structure in the speed when the data is getting bigger and bigger.
That's it. Test done!
