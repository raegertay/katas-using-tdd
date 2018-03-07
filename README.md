# Katas Using TDD
Solving katas on Codewars using TDD

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
It took me abit of time to come up with the idea of using `#slice!` to extract the smaller arrays from the bigger `work` array.
