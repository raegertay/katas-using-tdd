# Katas Using TDD
My learning journey into TDD by solving code katas!

## 5 Steps to TDD*
1. Quickly add a test.
2. Run all tests and see the new one fail.
3. Make a little change.
4. Run all tests and see them all succeed.
5. Refactor to remove duplication.

**From "Test-Driven Development By Example" by Kent Beck*

## Kata #1
[Distribute server workload](https://www.codewars.com/kata/distribute-server-workload/train/ruby)
(6 kyu) 

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
(6 kyu)

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

## Kata #3
[Mumbling](https://www.codewars.com/kata/mumbling/train/ruby)
(7 kyu)

My solution:
```ruby
def accum(s)
  s.chars.map.with_index { |c, i| (c * (i+1)).capitalize }.join('-')
end
```

My tests:
```ruby
Test.describe 'Given one letter' do
  it 'should return me the letter capitalized' do
    Test.assert_equals(accum('a'), 'A')
  end
end

Test.describe 'Given two letters' do
  it 'should return me the correct output' do
    Test.assert_equals(accum('ab'), 'A-Bb')
  end
end

Test.describe 'Given three letters' do
  it 'should return me the correct output' do
    Test.assert_equals(accum('abc'), 'A-Bb-Ccc')
  end
end
```

Reflection:

I feel that TDD isn't really neccessary for this kata. Reason being after the third test, I realised I had to make major changes my code in order to generalize for the scenario, which I could have coded from the start. I guess for easier kata, TDD isn't really worth the effort.

## Kata #4
[Set - the card game](https://www.codewars.com/kata/set-the-card-game/train/ruby)
(4 kyu)

My solution:
```ruby
def get_solutions( cards )
  p cards
  positions = [0,1,2].repeated_permutation(2).to_a + [[3,0], [3,1], [3,2]]
  possible_combinations = positions.combination(3).to_a
  solutions = []
  5.times do
    possible_combinations.shuffle!
    used_positions = []
    solutions << possible_combinations.select do |card_positions|
      cards_unused = true
      card_positions.each do |position|  
        cards_unused = false if used_positions.include?(position)
      end
      cards_combination = card_positions.map { |position| cards[position[0]][position[1]] }
      if is_a_set?(cards_combination) && cards_unused
        used_positions += card_positions
        true
      end
    end
  end
  max_solution_length = solutions.map(&:length).max
  solutions.find { |solution| solution.length == max_solution_length }
end

def is_a_set?(cards)
  %i[count color shape filling].all? do |attribute|
    values = cards.map(&attribute)
    same?(values) || unique?(values)
  end
end

def same?(values)
  reference_value = values[0]
  values.all? { |value| value == reference_value }
end

def unique?(values)
  values.uniq.length == values.length
end
```

My tests:
```ruby
describe '#same?' do
  it 'should return true if all values in the input array are the same' do
    Test.assert_equals(same?([1,1,1]), true)
  end
  it 'should return false if values in the input array are different' do
    Test.assert_equals(same?([1,2,1]), false)
  end
end

describe '#unique?' do
  it 'should return true if all values in the input array are unqiue' do
    Test.assert_equals(unique?([1,2,3]), true)
  end
  it 'should return false if values in the input array are not unique' do
    Test.assert_equals(unique?([1,2,1]), false)
  end
end

# Card = Struct.new(:count, :color, :shape, :filling)
card_1 = Card.new(1, 2, 1, 3)
card_2 = Card.new(1, 2, 2, 2)
card_3 = Card.new(1, 2, 3, 1)
card_4 = Card.new(2, 2, 1, 1)

describe '#is_a_set?' do
  it 'should return true if the 3 cards form a set' do
    Test.assert_equals(is_a_set?([card_1, card_2, card_3]), true)
  end  
  it 'should return false if the 3 cards do not form a set' do
    Test.assert_equals(is_a_set?([card_1, card_2, card_4]), false)
  end
end

hand = "1212:1113:1111:2222:2113:2111:3112:3113:3111:1222:1223:2211"
cards_attributes = hand.split(':')
cards = [[], [], [], []]
index = 0
4.times do |x|
  3.times do |y|
    attr = cards_attributes[index]
    cards[x][y] = Card.new(attr[0], attr[1], attr[2], attr[3])
    index += 1
  end
end

p get_solutions(cards)
```

Reflection:

The toughest kata I had solved so far. It started pretty well when I used TDD to help me code the helper methods. However, when it comes to the main method, I start to find TDD a chore as the it is troublesome to come up with the input data. As such, I decided to give up TDD at this point and went straight to writing the code. I was doing pretty well until I got stuck at a particular part and I had to write the code to generate the data for that particular test. Anyway, even though I didn't use TDD for the whole kata, it is definitely still useful for part of the solution.

## Kata #5
[Simple Events](https://www.codewars.com/kata/simple-events/train/ruby)
(5 kyu)

My solution:
```ruby
class Event
  attr_reader :functions
  
  def initialize
    @functions = []
  end
  
  def subscribe(function)
    @functions << function
  end
  
  def unsubscribe(function)
    @functions.delete(function)
  end
  
  def emit(*args)
    functions.each { |f| f.call(*args) }
  end
end
```

My tests:
```ruby
class Testf
  attr_accessor :calls, :args
  def initialize
    @calls=0
    @args=[]
  end
  def call(*args)
    @calls+=1
    @args+=args
  end
end

describe 'Event' do
  it 'should have a .subscribe() method, which takes a function and stores it as its handler' do
    event = Event.new
    f=Testf.new
    event.subscribe(f)
    Test.assert_equals(event.functions[0], f)
  end
  
  it 'should have an .unsubscribe() method, which takes a function and removes it from its handlers' do
    event = Event.new
    f=Testf.new
    event.subscribe(f)
    event.unsubscribe(f)
    Test.assert_equals(event.functions.length, 0)
  end
  
  it 'should have an .emit() method, which takes an arbitrary number of arguments and calls all the stored functions with these arguments' do
    event = Event.new
    f=Testf.new
    event.subscribe(f)
    args = [1, 'foo', true]
    event.emit(*args)
    Test.assert_equals(f.args, args)
  end
end
```

Reflection:

I feel this is a very straightforward kata to use TDD. The key to note is that one would need to create an attribute reader for the stored functions in order to have something to assert to. I learnt that sometimes its necessary to add certain api to the class in order to make testing easier.

## Kata #6
[80's Kids #10: Captain Planet](https://www.codewars.com/kata/568eeb1ce6f9e820c800000b/train/ruby)
(4 kyu)

My solution:
```ruby
def parse_data
  data = $data_file
  locations = data.scan(/Location: (\w+)/).flatten
  am_levels = data.scan(/Ammonia: (\d+)/).flatten.map(&:to_i)
  no_levels = data.scan(/Nitrogen Oxide: (\d+)/).flatten
  co_levels = data.scan(/Carbon Monoxide: (\d+)/).flatten
  
  highest_am_locations = []
  highest_no_locations = []
  highest_co_locations = []
  
  am_levels.each.with_index do |level, index|
    highest_am_locations << locations[index] if level == am_levels.max
  end
  
  no_levels.each.with_index do |level, index|
    highest_no_locations << locations[index] if level == no_levels.max
  end
  
  co_levels.each.with_index do |level, index|
    highest_co_locations << locations[index] if level == co_levels.max
  end
  
  "Ammonia levels in #{highest_am_locations.join(', ')} are too high. "\
  "Nitrogen Oxide levels in #{highest_no_locations.join(', ')} are too high. "\
  "Carbon Monoxide levels in #{highest_co_locations.join(', ')} are too high."
end
```

My tests:
```ruby
# For testing purpose, allow .parse_data() to accept one argument to pass in our data

one_location_data = "##################################\n"\
          "Location: DEU\n"\
          "##################################\n"\
          "Ammonia: 023 particles\n"\
          "Nitrogen Oxide: 919 particles\n"\
          "Carbon Monoxide: 027 particles\n"\
          "##################################\n"
          


describe "One location only" do
  it "should report the same location for all gases" do
    Test.assert_equals(parse_data(one_location_data), "Ammonia levels in DEU are too high. Nitrogen Oxide levels in DEU are too high. Carbon Monoxide levels in DEU are too high.")
  end
end

describe "Multiple locations" do
  it "should report the location with the highest gas level for all gases" do
    Test.assert_equals(parse_data($data_file), "Ammonia levels in USA, CHN are too high. Nitrogen Oxide levels in DEU are too high. Carbon Monoxide levels in AUS, BHS, CRI are too high.")
  end
end
```

My reflection:

I only have 2 tests for this kata due to the complexity of preparing the test data. Even so, the first test proves to be extremely useful as it gave me the idea to use #scan and regexp to solve this kata. The idea came about because I was looking for the fastest way to pass the first test. Another thing to note is that I had to tweak the signature of the #parse_data method to accept the test data I prepared while testing. Overall, this kata taught me to never underestimate the importance of the basic test cases because it might give us the ideas to solve the kata.

## Kata #7
[Roman Numerals Decoder](https://www.codewars.com/kata/roman-numerals-decoder/train/ruby)
(4 kyu)

My solution:
```ruby
ROMAN_VALUES = {
  'I' => 1,
  'V' => 5,
  'X' => 10,
  'L' => 50,
  'C' => 100,
  'D' => 500,
  'M' => 1000
}.freeze

NEXT_CHAR_SET = {
  'I' => /[VX]/,
  'X' => /[LC]/,
  'C' => /[DM]/
}.freeze

def solution(roman)
  decimal = 0
  roman.each_char.with_index do |c, i|
    next_char = roman[i+1]
    decimal += roman_value( c, subtracted?(next_char, NEXT_CHAR_SET[c]) )
  end
  decimal
end

private

def subtracted?(next_char, next_char_set)
  return false unless next_char_set
  next_char && next_char.match(next_char_set)
end

def roman_value(roman, subtracted=false)
  subtracted ? -ROMAN_VALUES[roman] : ROMAN_VALUES[roman]
end
```

My tests:
```ruby
require "minitest/autorun"

describe 'solution' do
  it "should return 1 for 'I'" do
    solution('I').must_equal 1
  end
  
  it "should return 2 for 'II'" do
    solution('II').must_equal 2
  end

  it "should return 3 for 'III'" do
    solution('III').must_equal 3
  end

  it "should return 4 for 'IV'" do
    solution('IV').must_equal 4
  end

  it "should return 5 for 'V'" do
    solution('V').must_equal 5
  end

  it "should return 6 for 'VI'" do
    solution('VI').must_equal 6
  end

  it "should return 7 for 'VII'" do
    solution('VII').must_equal 7
  end

  it "should return 9 for 'IX'" do
    solution('IX').must_equal 9
  end

  it "should return 10 for 'X'" do
    solution('X').must_equal 10
  end

  it "should return 14 for 'XL'" do
    solution('XL').must_equal 40
  end
end
```

My Reflection:

For this kata, I find it really hard to refactor and generalize my code after making the test case pass. For example, for the roman number "IV", it took me quite a while to get the logic to make the "I" a subtracted number instead of addition. However, after I saw the top solution in codewars, I realized I could have just hard coded "IV" as 4, which make the code much easier to write and understand. Basically, I generalized my solution the wrong way and it became over-engineered. This kata taught me to be careful of refactoring too much.
