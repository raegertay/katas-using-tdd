# Katas Using TDD
My learning journey into TDD by solving code katas!

## Kata #1
[Distribute server workload](https://www.codewars.com/kata/distribute-server-workload/train/ruby)

My solution:
```ruby
def distribute(nodes, workload)
  work_per_node = workload / nodes
  extra_work = workload % nodes
  work = (0...workload).to_a
  work_distribution = []
  extra_work.times { work_distribution <<  work.slice!(0, work_per_node+1) } 
  (nodes-extra_work).times { work_distribution << work.slice!(0, work_per_node) }
  work_distribution
end
```

My tests:
```ruby
describe "Even workload" do
  it "should distribute workload equally among nodes" do
    Test.assert_equals(distribute(1, 1), [[0]])
    Test.assert_equals(distribute(3, 3), [[0], [1], [2]])
    Test.assert_equals(distribute(2, 4), [[0, 1], [2, 3]])
    Test.assert_equals(distribute(3, 9), [[0, 1, 2], [3, 4, 5], [6, 7, 8]])
  end
end
  
describe 'Uneven workload' do
  it "should distribute extra work starting from first server" do
    Test.assert_equals(distribute(2, 5), [[0, 1, 2], [3, 4]])
    Test.assert_equals(distribute(4, 10), [[0, 1, 2], [3, 4, 5], [6, 7], [8, 9]])
    Test.assert_equals(distribute(4, 5), [[0, 1], [2], [3], [4]])
  end
end
```

Reflection:

The most difficult part of this kata is figuring out how to get the nested arrays when the workload is greater than nodes.
I had to experiment with a few different methods to figure out that I can use `#slice!` to extract the smaller arrays from the bigger `work` array. This kata taught me the importance of experimenting around when stuck.

## Kata #2
[Simple Fun #332: Catch Thief](https://www.codewars.com/kata/simple-fun-number-332-catch-thief/train/ruby)

My solution:
```ruby
def catch_thief(queue)
  num_of_thieves = 0
  queue.each_char.with_index do |c, i|
    if c.match(/\d/)
      range = c.to_i
      start_index = ( i-range < 0 ? 0 : i-range )
      end_index = i + range
      people_around_police = queue[start_index..end_index]
      num_of_thieves += people_around_police.count('X')
      queue = queue[0...start_index] + people_around_police.gsub('X', '#') + (queue[(end_index+1)..-1] || '')
    end
  end
  num_of_thieves
end
```

My tests:
```ruby
Test.describe 'One police with watching range of 1' do
  it 'should only catch thief beside the police' do
    Test.assert_equals(catch_thief("1X"), 1)
    Test.assert_equals(catch_thief("1#X"), 0)
    Test.assert_equals(catch_thief("X1"), 1)
  end
end

Test.describe 'One police with watching range of 2' do
  it 'should only catch thief within 2 spaces from the police' do
    Test.assert_equals(catch_thief("XX2XX"), 4)
  end
end

Test.describe 'Two police' do
  it 'should only catch each thief once' do
    Test.assert_equals(catch_thief("1X1"), 1)
  end
end

Test.describe "Default tests" do
  Test.assert_equals(catch_thief("X1X#2X#XX"),3)
  Test.assert_equals(catch_thief("X5X#3X###XXXX##1#X1X"),5)
  Test.assert_equals(catch_thief("X#X1#X9XX"),5)
end
```

Reflection:

This is one of those kata where I had no clue at all on how to approach the problem. I had to start from the really simple scenario (i.e 'One police with watching range of 1') and progressively add more complex test scenarios. However, after a certain point (i.e 'Two police'), everything just starts falling into the place. And before I knew it, I've got the solution. This kata taught me that the importance of progressively adding complexity to the test cases.
