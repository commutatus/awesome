---
title: Rubocop
nav_order: 10
parent: Efficient
grand_parent: Engineering
---

These are standards that Rails applications should have in terms of code style and performance. To check those standards RuboCop is used.

# Rubocop

1. Add these three gems to your `Gemfile` to support RuboCop in your project.

		gem 'rubocop', require: false
		gem 'rubocop-rails', require: false
		gem 'rubocop-performance', require: false								

2. Create a file called `.rubocop.yml` in your root folder of your project

3. Add the following code to the file which we created called `.rubocop.yml`, currently the given config file will only check the styles.

	```yaml
	AllCops:
	  DisabledByDefault: true 																																																						
	Style/Alias:
	  Description: 'Use alias_method instead of alias.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#alias-method'
	  Enabled: true 																																											
	Style/ArrayJoin:
		  Description: 'Use Array#join instead of Array#*.'
		  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#array-join'
		  Enabled: true 																														
	Style/AsciiComments:
	  Description: 'Use only ascii symbols in comments.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#english-comments'
	  Enabled: true 																															
	Style/Attr:
	  Description: 'Checks for uses of Module#attr.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#attr'
	  Enabled: true 																															
	Style/CaseEquality:
	  Description: 'Avoid explicit use of the case equality operator(===).'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-case-equality'
	  Enabled: true 																															
	Style/CharacterLiteral:
	  Description: 'Checks for uses of character literals.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-character-literals'
	  Enabled: true 																															
	Style/ClassAndModuleChildren:
	  Description: 'Checks style of children classes and modules.'
	  Enabled: true
	  EnforcedStyle: nested 																												
	Style/ClassVars:
	  Description: 'Avoid the use of class variables.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-class-vars'
	  Enabled: true 																																			
	Style/CollectionMethods:
	  Enabled: true
	  PreferredMethods:
	    find: detect
	    inject: reduce
	    collect: map
	    find_all: select 																																		
	Style/ColonMethodCall:
	  Description: 'Do not use :: for method call.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#double-colons'
	  Enabled: true 																																
	Style/CommentAnnotation:
	  Description: >-
	                 Checks formatting of special comments
	                 (TODO, FIXME, OPTIMIZE, HACK, REVIEW).
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#annotate-keywords'
	  Enabled: true 																															
	Style/PreferredHashMethods:
	  Description: 'Checks use of `has_key?` and `has_value?` Hash methods.'
	  StyleGuide: '#hash-key'
	  Enabled: true 																																							
	Style/Documentation:
	  Description: 'Document classes and non-namespace modules.'
	  Enabled: true 																																
	Style/DoubleNegation:
	  Description: 'Checks for uses of double negation (!!).'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-bang-bang'
	  Enabled: true 																																				
	Style/EachWithObject:
	  Description: 'Prefer `each_with_object` over `inject` or `reduce`.'
	  Enabled: true 																																
	Style/EmptyLiteral:
	  Description: 'Prefer literals to Array.new/Hash.new/String.new.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#literal-array-hash'
	  Enabled: true 																															
	Style/EvenOdd:
	  Description: 'Favor the use of Fixnum#even? && Fixnum#odd?'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#predicate-methods'
	  Enabled: true 																															
	Style/FormatString:
	  Description: 'Enforce the use of Kernel#sprintf, Kernel#format or String#%.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#sprintf'
	  Enabled: true 																																
	Style/GlobalVars:
	  Description: 'Do not introduce global variables.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#instance-vars'
	  Reference: 'http://www.zenspider.com/Languages/Ruby/QuickRef.html'
	  Enabled: true 																																
	Style/GuardClause:
	  Description: 'Check for conditionals that can be replaced with guard clauses'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-nested-conditionals'
	  Enabled: true 																															
	Style/IfUnlessModifier:
	  Description: >-
	                 Favor modifier if/unless usage when you have a
	                 single-line body.
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#if-as-a-modifier'
	  Enabled: true 																															
	Style/IfWithSemicolon:
	  Description: 'Do not use if x; .... Use the ternary operator instead.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-semicolon-ifs'
	  Enabled: true 																															
	Style/InlineComment:
	  Description: 'Avoid inline comments.'
	  Enabled: true 																															
	Style/Lambda:
	  Description: 'Use the new lambda literal syntax for single-line blocks.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#lambda-multi-line'
	  Enabled: true 																																	
	Style/LambdaCall:
	  Description: 'Use lambda.call(...) instead of lambda.(...).'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#proc-call'
	  Enabled: true 																																
	Style/LineEndConcatenation:
	  Description: >-
	                 Use \ instead of + or << to concatenate two string literals at
	                 line end.
	  Enabled: true 																															
	Style/ModuleFunction:
	  Description: 'Checks for usage of `extend self` in modules.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#module-function'
	  Enabled: true 																															
	Style/MultilineBlockChain:
	  Description: 'Avoid multi-line chains of blocks.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#single-line-blocks'
	  Enabled: true 																															
	Style/NegatedIf:
	  Description: >-
	                 Favor unless over if for negative conditions
	                 (or control flow or).
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#unless-for-negatives'
	  Enabled: true 																																	
	Style/NegatedWhile:
	  Description: 'Favor until over while for negative conditions.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#until-for-negatives'
	  Enabled: true 																																
	Style/Next:
	  Description: 'Use `next` to skip iteration instead of a condition at the end.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-nested-conditionals'
	  Enabled: true 																																		
	Style/NilComparison:
	  Description: 'Prefer x.nil? to x == nil.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#predicate-methods'
	  Enabled: true 																																			
	Style/Not:
	  Description: 'Use ! instead of not.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#bang-not-not'
	  Enabled: true 																															
	Style/NumericLiterals:
	  Description: >-
	                 Add underscores to large numeric literals to improve their
	                 readability.
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#underscores-in-numerics'
	  Enabled: true 																																		
	Style/OneLineConditional:
	  Description: >-
	                 Favor the ternary operator(?:) over
	                 if/then/else/end constructs.
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#ternary-operator'
	  Enabled: true 																																		
	Style/PercentLiteralDelimiters:
	  Description: 'Use `%`-literal delimiters consistently'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#percent-literal-braces'
	  Enabled: true 																															
	Style/PerlBackrefs:
	  Description: 'Avoid Perl-style regex back references.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-perl-regexp-last-matchers'
	  Enabled: true 																															
	Style/Proc:
	  Description: 'Use proc instead of Proc.new.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#proc'
	  Enabled: true 																															
	Style/RaiseArgs:
	  Description: 'Checks the arguments passed to raise/fail.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#exception-class-messages'
	  Enabled: true 																																	
	Style/RegexpLiteral:
	  Description: 'Use / or %r around regular expressions.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#percent-r'
	  Enabled: true 																															
	Style/Sample:
	  Description: >-
	    Use `sample` instead of `shuffle.first`,
	    `shuffle.last`, and `shuffle[Fixnum]`.
	  Reference: 'https://github.com/JuanitoFatas/fast-ruby#arrayshufflefirst-vs-arraysample-code'
	  Enabled: true 																																		
	Style/SingleLineBlockParams:
	  Description: 'Enforces the names of some block params.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#reduce-blocks'
	  Enabled: true 																																		
	Style/SingleLineMethods:
	  Description: 'Avoid single-line methods.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-single-line-methods'
	  Enabled: true 																																						
	Style/SignalException:
	  Description: 'Checks for proper usage of fail and raise.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#fail-method'
	  Enabled: true 																																		
	Style/SpecialGlobalVars:
	  Description: 'Avoid Perl-style global variables.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-cryptic-perlisms'
	  Enabled: true 																															
	Style/StringLiterals:
	  Description: 'Checks if uses of quotes match the configured preference.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#consistent-string-literals'
	  EnforcedStyle: double_quotes
	  Enabled: true 																																
	Style/TrailingCommaInArguments:
	  Description: 'Checks for trailing comma in argument lists.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-trailing-array-commas'
	  EnforcedStyleForMultiline: comma
	  SupportedStylesForMultiline:
	    - comma
	    - consistent_comma
	    - no_comma
	  Enabled: true 																																			
	Style/TrailingCommaInArrayLiteral:
	  Description: 'Checks for trailing comma in array literals.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-trailing-array-commas'
	  EnforcedStyleForMultiline: comma
	  SupportedStylesForMultiline:
	    - comma
	    - consistent_comma
	    - no_comma
	  Enabled: true 																															
	Style/TrailingCommaInHashLiteral:
	  Description: 'Checks for trailing comma in hash literals.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#no-trailing-array-commas'
	  EnforcedStyleForMultiline: comma
	  SupportedStylesForMultiline:
	    - comma
	    - consistent_comma
	    - no_comma
	  Enabled: true 																																				
	Style/TrivialAccessors:
	  Description: 'Prefer attr_* methods to trivial readers/writers.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#attr_family'
	  Enabled: true 																															
	Style/VariableInterpolation:
	  Description: >-
	                 Don't interpolate global, instance and class variables
	                 directly in strings.
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#curlies-interpolate'
	  Enabled: true 																																	
	Style/WhenThen:
	  Description: 'Use when x then ... for one-line cases.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#one-line-cases'
	  Enabled: true 																																							
	Style/WhileUntilModifier:
	  Description: >-
	                 Favor modifier while/until usage when you have a
	                 single-line body.
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#while-as-a-modifier'
	  Enabled: true 																																
	Style/WordArray:
	  Description: 'Use %w or %W for arrays of words.'
	  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#percent-w'
	  Enabled: true
	```


4. Now in the terminal we can check the number of [offences](https://rubocop.readthedocs.io/en/latest/cops/) which we did in our code base by running the command

		$ rubocop

5. Alternatively we can pass RuboCop a list of files and directories to check

		$ rubocop app spec lib/something.rb

6. You can also run RuboCop in an auto-correct mode, where it will try to automatically fix the problems it found in your code

		$ rubocop -a

7. Additionally to check coffee and scss files, add this two gems to your file

		gem 'scss_lint', require: false
		gem 'coffeelint'

8. Now run a command to check the sass files

		$	scss-lint

	Also you can run the command to check specific files using
		
		$	scss-lint app/assets/stylesheets/

9. To check coffee lint use the command

		$	bundle exec rake coffeelint
