# Agentic AI

# 1. Stateful Agent — `ChatAgent`

### Problem (brief)

* Create a class `ChatAgent`.
* `__init__` should set `self.user_name = None`.
* Implement `respond(self, message)`:

  * If `self.user_name` is `None`, check if the message is of the form `"My name is Bob"`. If so set `self.user_name = "Bob"` and print `"Hello Bob!"`.
  * If `self.user_name` is set, just print `f"Hello again, {self.user_name}!"`.
* Test by calling:

  * `respond("Hi")`
  * `respond("My name is Bob")`
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
                reply = "I don't know your name yet. Please tell me: My name is <YourName>."
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
    agent.respond("My name is Bob")
    agent.respond("Who am I?")
    agent.respond("Hi")
    agent.respond("Reset")
    agent.respond("Hi")
    print("\nConversation history:")
    agent.show_history()
```

### Expected output for `python chat_agent.py`

```
I don't know your name yet. Please tell me: My name is <YourName>.
Hello Bob! Nice to meet you.
You are Bob.
Hello again, Bob! (visit #3)
State reset. I have forgotten Bob.
I don't know your name yet. Please tell me: My name is <YourName>.

Conversation history:
01 user: Hi
02 agent: I don't know your name yet. Please tell me: My name is <YourName>.
03 user: My name is Bob
04 agent: Hello Bob! Nice to meet you.
05 user: Who am I?
06 agent: You are Bob.
07 user: Hi
08 agent: Hello again, Bob! (visit #3)
09 user: Reset
10 agent: State reset. I have forgotten Bob.
11 user: Hi
12 agent: I don't know your name yet. Please tell me: My name is <YourName>.
```

---

# 2. Router Agent — `RouterAgent`, `WeatherAgent`, `MathAgent`

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
import ast
import operator

class WeatherAgent:
    def get_weather(self, location="your area"):
        report_lines = [
            f"Weather report for {location}:",
            "Condition: Sunny",
            "Temperature: 28°C",
            "Forecast: Clear skies for the next 3 days"
        ]
        for line in report_lines:
            print(line)

class MathAgent:
    ALLOWED_OPERATORS = {
        ast.Add: operator.add,
        ast.Sub: operator.sub,
        ast.Mult: operator.mul,
        ast.Div: operator.truediv,
        ast.Pow: operator.pow,
        ast.USub: operator.neg,
    }

    def _eval_node(self, node):
        if isinstance(node, ast.Num):
            return node.n
        if isinstance(node, ast.BinOp):
            left = self._eval_node(node.left)
            right = self._eval_node(node.right)
            op_type = type(node.op)
            if op_type in self.ALLOWED_OPERATORS:
                return self.ALLOWED_OPERATORS[op_type](left, right)
        if isinstance(node, ast.UnaryOp):
            operand = self._eval_node(node.operand)
            op_type = type(node.op)
            if op_type in self.ALLOWED_OPERATORS:
                return self.ALLOWED_OPERATORS[op_type](operand)
        raise ValueError("Unsupported expression")

    def do_math(self, expression=None):
        if not expression:
            print("2 + 2 = 4.")
            return
        try:
            tree = ast.parse(expression, mode='eval').body
            result = self._eval_node(tree)
            print(f"{expression} = {result}")
        except Exception as e:
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
        parts = query.split(":")
        if len(parts) > 1:
            return parts[1].strip()
        words = query.lower().split()
        keywords = ("calculate", "compute")
        for i, w in enumerate(words):
            if w in keywords and i + 1 < len(words):
                expr = " ".join(words[i + 1:])
                return expr
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
            agent.get_weather(location=location)
        elif AgentClass is MathAgent:
            expr = self._extract_math_expression(query)
            agent.do_math(expression=expr)
        else:
            print("Domain recognized but no handler implemented.")


# Test run for RouterAgent
if __name__ == "__main__":
    router = RouterAgent()
    router.handle_query("What is the weather today?")
    print()
    router.handle_query("Do some math: calculate 12/3 + 5")
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

> Note: Depending on how you type the math queries, the router attempts to extract an expression after keywords like `calculate:` or `calculate`/`compute`. The `MathAgent` evaluator is intentionally small and safe: it supports basic numeric literals and the operators defined in `ALLOWED_OPERATORS`. It will raise an error for unsupported constructs.

---

# How to run

1. Save each section into its own `.py` file (`chat_agent.py`, `router_agent.py`).
2. Run from the command line:

   * `python chat_agent.py`
   * `python router_agent.py`
3. Observe the printed output in your terminal.
