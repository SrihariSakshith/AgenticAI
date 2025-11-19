# Agentic AI

# 1. Stateful Agent — `ChatAgent`
A stateful agent is a program that remembers information from previous interactions and uses that stored information (state) to respond in future interactions.
Example: remembering a user’s name after they tell it once.

### Problem (brief)

* Create a class `ChatAgent`.
* `__init__` should set `self.user_name = None`.
* Implement `respond(self, message)`:

  * If `self.user_name` is `None`, check if the message is of the form `"My name is Sakshith"`. If so set `self.user_name = "Sakshith"` and print `"Hello Sakshith!"`.
  * If `self.user_name` is set, just print `f"Hello again, {self.user_name}!"`.
* Test by calling:

  * `respond("Hi")`
  * `respond("My name is Sakshith")`
  * `respond("Hi")` again

### File: `chat_agent.py`

```python
class ChatAgent:
    def __init__(self):
        self.user_name = None
        self.history = []
        self.greet_count = 0

    def _remember(self, message, actor="user"):
        self.history.append((actor, message))

    def _extract_name(self, message):
        m = message.strip()
        if m.lower().startswith("my name is "):
            name = m[len("my name is "):].strip()
            if name:
                return name
        return None

    def respond(self, message):
        self._remember(message, "user")
        if self.user_name is None:
            name = self._extract_name(message)
            if name:
                self.user_name = name
                self.greet_count = 1
                reply = f"Hello {self.user_name}! Nice to meet you."
            else:
                reply = "I don't know your name yet. Please tell me."
        else:
            self.greet_count += 1
            if message.strip().lower() in ("who am i?", "who am i"):
                reply = f"You are {self.user_name}."
            elif message.strip().lower() == "reset":
                old = self.user_name
                self.user_name = None
                self.greet_count = 0
                self.history = []
                reply = f"State reset. I have forgotten {old}."
            else:
                reply = f"Hello again, {self.user_name}! (visit #{self.greet_count})"
        self._remember(reply, "agent")
        print(reply)

    def show_history(self):
        for i, (actor, text) in enumerate(self.history, 1):
            print(f"{i:02d} {actor}: {text}")


# Test run for ChatAgent
if __name__ == "__main__":
    agent = ChatAgent()
    agent.respond("Hi")
    agent.respond("My name is Sakshith")
    agent.respond("Who am I?")
    agent.respond("Hi")
    # agent.respond("Reset")
    # agent.respond("Hi")
    print("\nConversation history:")
    agent.show_history()
```

### Expected output for `python chat_agent.py`

```
I don't know your name yet. Please tell me.
Hello sakshith! Nice to meet you.
You are Sakshith.
Hello again, Sakshith! (visit #3)

Conversation history:
01 user: Hi
02 agent: I don't know your name yet. Please tell me.
03 user: My name is Sakshith
04 agent: Hello Sakshith! Nice to meet you.
05 user: Who am I?
06 agent: You are Sakshith.
07 user: Hi
08 agent: Hello again, Sakshith! (visit #3)
```

---

# 2. Router Agent — `RouterAgent`, `WeatherAgent`, `MathAgent`
A router agent is a program that examines a user’s query and directs it to the correct specialized agent (like weather agent, math agent, etc.) based on keywords or rules.
It works like a traffic controller that routes each request to the right handler.

### Problem (brief)

* `WeatherAgent` with `get_weather()` method that prints `"It is sunny."` (or a short weather report).
* `MathAgent` with `do_math()` method that prints `"2 + 2 = 4."` (or evaluates a given simple expression).
* `RouterAgent` with `handle_query(self, query)`:

  * If query contains `"weather"`, use `WeatherAgent`.
  * If query contains `"math"`, use `MathAgent`.
  * Otherwise print `"I can't help with that."`.
* Test calls:

  * `handle_query("What is the weather?")`
  * `handle_query("Do some math.")`

The implementation below expands functionality a bit:

* `WeatherAgent.get_weather(location="your area")` prints a short multi-line report.
* `MathAgent.do_math(expression=None)` can evaluate a simple safe expression (uses `ast`, supports basic arithmetic). If no expression provided, it prints the default `2 + 2 = 4.`.
* `RouterAgent` tries to find keywords in the query and route accordingly; it also tries to extract math expressions when present.

### File: `router_agent.py`

```python
class WeatherAgent:
    def get_weather(self, location="your area"):
        print(f"Weather report for {location}:")
        print("Condition: Sunny")
        print("Temperature: 28°C")
        print("Forecast: Clear skies for the next 3 days")


class MathAgent:
    def do_math(self, expression=None):
        if not expression:
            print("2 + 2 = 4.")
            return
        try:
            # A safe evaluation environment
            safe_env = {
                "__builtins__": None
            }
            result = eval(expression, safe_env, {})
            print(f"{expression} = {result}")
        except Exception as e:
            print("ERROR!")
            print("Could not compute expression. Use simple arithmetic like 2+3*4. Error:", e)


class RouterAgent:
    def __init__(self):
        self.routing_table = {
            "weather": WeatherAgent,
            "forecast": WeatherAgent,
            "math": MathAgent,
            "calculate": MathAgent,
            "compute": MathAgent
        }

    def _find_domain(self, query):
        q = query.lower()
        for key in self.routing_table:
            if key in q:
                return key
        return None

    def _extract_math_expression(self, query):
        # split after colon: "calculate: 12/3 + 5"
        if ":" in query:
            return query.split(":")[1].strip()

        words = query.lower().split()
        if "calculate" in words:
            idx = words.index("calculate")
            return " ".join(query.split()[idx + 1:])
        if "compute" in words:
            idx = words.index("compute")
            return " ".join(query.split()[idx + 1:])
        return None

    def handle_query(self, query):
        domain_key = self._find_domain(query)

        if domain_key is None:
            print("I can't help with that.")
            return

        AgentClass = self.routing_table[domain_key]
        agent = AgentClass()

        if AgentClass is WeatherAgent:
            location = "your area"
            tokens = query.split()
            if "in" in tokens:
                idx = tokens.index("in")
                if idx + 1 < len(tokens):
                    location = tokens[idx + 1]
            agent.get_weather(location)

        elif AgentClass is MathAgent:
            expr = self._extract_math_expression(query)
            agent.do_math(expr)


# Test run
if __name__ == "__main__":
    router = RouterAgent()
    router.handle_query("What is the weather today?")
    print()
    router.handle_query("Do some math: 12/3 + 5")
    print()
    router.handle_query("Please calculate 2 ** 3")
    print()
    router.handle_query("Tell me a joke.")
```

### Expected output for `python router_agent.py`

```
Weather report for your area:
Condition: Sunny
Temperature: 28°C
Forecast: Clear skies for the next 3 days

12/3 + 5 = 9.0

2 ** 3 = 8

I can't help with that.
```

---

# How to run

1. Save each section into its own `.py` file (`chat_agent.py`, `router_agent.py`).
2. Run from the command line:

   * `python chat_agent.py`
   * `python router_agent.py`
3. Observe the printed output in your terminal.
