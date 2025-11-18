# AgenticAI
1.	Stateful Agent: Write a Python program.
o	Create a class ChatAgent.
o	Inside its __init__, give it a "state" variable: self.user_name = None.
o	Create a respond(self, message) method.
o	Inside this method, if self.user_name is None, check if the message is "My name is Bob". If it is, set self.user_name = "Bob" and print "Hello Bob!".
o	If self.user_name is set, just print f"Hello again, {self.user_name}!".
o	Test it by calling respond("Hi"), then respond("My name is Bob"), then respond("Hi") again.
2.	Router Agent (Graph Logic): Write a Python program.
o	Create a WeatherAgent class with a get_weather() method that prints "It is sunny."
o	Create a MathAgent class with a do_math() method that prints "2 + 2 = 4."
o	Create a RouterAgent class with a handle_query(self, query) method.
o	Inside handle_query, if the query contains "weather", create and use the WeatherAgent. If the query contains "math", create and use the MathAgent. Otherwise, print "I can't help with that."
o	Test it by calling handle_query("What is the weather?") and handle_query("Do some math.").

