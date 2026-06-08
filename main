"""
CodeAlpha Internship — AI Chatbot (Production-Grade)
Uses Anthropic Claude API for real intelligent responses.
Features: memory, context, personality, graceful exit.
"""

import os
import sys
import json
import urllib.request
import urllib.error
from datetime import datetime

# ──────────────────────────────────────────────
# CONFIG
# ──────────────────────────────────────────────

def load_config() -> dict:
    """Load config from .env file or fall back to defaults."""
    config = {
        "BOT_NAME": "AlphaBot",
        "USER_NAME": "User",
        "API_KEY": "",
        "MODEL": "claude-sonnet-4-20250514",
        "MAX_TOKENS": "1000",
        "MAX_HISTORY": "20",
    }
    if os.path.exists(".env"):
        try:
            with open(".env", "r", encoding="utf-8") as f:
                for line in f:
                    line = line.strip()
                    if not line or line.startswith("#") or "=" not in line:
                        continue
                    key, val = line.split("=", 1)
                    config[key.strip()] = val.strip().strip('"').strip("'")
        except IOError:
            pass
    # Also check environment variables (overrides .env)
    for key in config:
        env_val = os.environ.get(key)
        if env_val:
            config[key] = env_val
    return config


# ──────────────────────────────────────────────
# ANTHROPIC API CALLER (no external libraries)
# ──────────────────────────────────────────────

def call_claude_api(api_key: str, model: str, system_prompt: str,
                    messages: list, max_tokens: int) -> str:
    """
    Call the Anthropic Messages API using only Python standard library.
    Returns the assistant's reply text, or an error string.
    """
    url = "https://api.anthropic.com/v1/messages"
    payload = {
        "model": model,
        "max_tokens": max_tokens,
        "system": system_prompt,
        "messages": messages,
    }
    data = json.dumps(payload).encode("utf-8")
    headers = {
        "Content-Type": "application/json",
        "x-api-key": api_key,
        "anthropic-version": "2023-06-01",
    }
    req = urllib.request.Request(url, data=data, headers=headers, method="POST")
    try:
        with urllib.request.urlopen(req, timeout=30) as resp:
            body = json.loads(resp.read().decode("utf-8"))
            return body["content"][0]["text"]
    except urllib.error.HTTPError as e:
        err_body = e.read().decode("utf-8")
        try:
            err_json = json.loads(err_body)
            return f"[API Error {e.code}]: {err_json.get('error', {}).get('message', err_body)}"
        except Exception:
            return f"[API Error {e.code}]: {err_body}"
    except urllib.error.URLError as e:
        return f"[Network Error]: {e.reason}. Check your internet connection."
    except Exception as e:
        return f"[Unexpected Error]: {e}"


# ──────────────────────────────────────────────
# RULE-BASED FALLBACK (works without API key)
# ──────────────────────────────────────────────

RULE_RESPONSES = {
    frozenset(["hello", "hi", "hey", "greetings", "howdy"]):
        "Hello! 👋 Great to meet you. What's on your mind?",
    frozenset(["how are you", "how's it going", "how are you doing", "you ok"]):
        "I'm running perfectly — no bugs today! 😄 How about you?",
    frozenset(["what is your name", "who are you", "your name", "identify yourself"]):
        None,  # handled dynamically with bot_name
    frozenset(["what can you do", "help", "commands", "capabilities"]):
        "I can chat, answer questions, tell jokes, give advice, and more! Just type anything.",
    frozenset(["tell me a joke", "joke", "make me laugh", "funny"]):
        "Why do programmers prefer dark mode? Because light attracts bugs! 🐛",
    frozenset(["what time is it", "current time", "time now"]):
        None,  # handled dynamically
    frozenset(["what is today", "today's date", "current date", "what day is it"]):
        None,  # handled dynamically
    frozenset(["thanks", "thank you", "thx", "ty"]):
        "You're welcome! 😊 Happy to help.",
    frozenset(["bye", "goodbye", "exit", "quit", "cya", "see you"]):
        None,  # handled as exit trigger
}

def rule_based_response(text: str, bot_name: str) -> str | None:
    """Return a hardcoded reply or None if no rule matches."""
    normalized = text.strip().lower()

    # Dynamic rules
    if any(w in normalized for w in ["your name", "who are you", "identify"]):
        return f"I'm {bot_name}, your AI-powered assistant! Nice to meet you. 🤖"
    if any(w in normalized for w in ["time", "clock"]) and "date" not in normalized:
        return f"Current time: {datetime.now().strftime('%I:%M %p')} ⏰"
    if any(w in normalized for w in ["date", "today", "day"]):
        return f"Today is {datetime.now().strftime('%A, %B %d, %Y')} 📅"

    for keyword_set, reply in RULE_RESPONSES.items():
        if any(kw in normalized for kw in keyword_set):
            return reply  # could be None for exit trigger

    return None  # no match → use AI or default fallback


# ──────────────────────────────────────────────
# TERMINAL UI HELPERS
# ──────────────────────────────────────────────

COLORS = {
    "reset":  "\033[0m",
    "bold":   "\033[1m",
    "cyan":   "\033[96m",
    "green":  "\033[92m",
    "yellow": "\033[93m",
    "red":    "\033[91m",
    "grey":   "\033[90m",
    "white":  "\033[97m",
    "blue":   "\033[94m",
}

def c(text: str, *codes: str) -> str:
    prefix = "".join(COLORS.get(code, "") for code in codes)
    return f"{prefix}{text}{COLORS['reset']}"

def print_banner(bot_name: str, has_api: bool):
    mode = c("AI Mode (Claude API)", "green", "bold") if has_api else c("Rule-Based Mode", "yellow", "bold")
    print(c("=" * 60, "cyan"))
    print(c(f"  🤖  {bot_name} — Intelligent Terminal Chatbot", "white", "bold"))
    print(c(f"  Mode: ", "grey") + mode)
    print(c(f"  Type 'help' for commands | 'bye' to exit", "grey"))
    print(c("=" * 60, "cyan"))

def print_bot(bot_name: str, msg: str):
    prefix = c(f"{bot_name}: ", "cyan", "bold")
    print(f"\n{prefix}{msg}")

def print_user_prompt(user_name: str) -> str:
    prompt = c(f"\n{user_name}: ", "green", "bold")
    try:
        return input(prompt)
    except EOFError:
        return "bye"

def print_typing():
    print(c("  [thinking...]", "grey"), end="\r", flush=True)

def clear_typing():
    print(" " * 20, end="\r", flush=True)

def print_separator():
    print(c("  " + "─" * 56, "grey"))


# ──────────────────────────────────────────────
# CHAT SESSION
# ──────────────────────────────────────────────

def build_system_prompt(bot_name: str, user_name: str) -> str:
    return (
        f"You are {bot_name}, a friendly, helpful, and witty AI chatbot. "
        f"You are talking to {user_name}. "
        "Keep responses concise (2-4 sentences max unless asked for more). "
        "Be warm, helpful, and occasionally use a relevant emoji. "
        "If asked to write code, format it properly. "
        f"Today's date is {datetime.now().strftime('%A, %B %d, %Y')}."
    )


class ChatSession:
    def __init__(self, config: dict):
        self.bot_name   = config["BOT_NAME"]
        self.user_name  = config["USER_NAME"]
        self.api_key    = config["API_KEY"]
        self.model      = config["MODEL"]
        self.max_tokens = int(config["MAX_TOKENS"])
        self.max_hist   = int(config["MAX_HISTORY"])
        self.history: list[dict] = []   # [{"role": ..., "content": ...}]
        self.has_api    = bool(self.api_key)
        self.system     = build_system_prompt(self.bot_name, self.user_name)
        self.msg_count  = 0

    def _trim_history(self):
        """Keep only the last N messages to avoid huge context."""
        if len(self.history) > self.max_hist:
            self.history = self.history[-self.max_hist:]

    def get_response(self, user_text: str) -> str:
        """
        Priority order:
        1. Rule-based (instant, offline) for common inputs
        2. Claude API (if API key available)
        3. Fallback default message
        """
        # Check rules first (fast)
        rule_reply = rule_based_response(user_text, self.bot_name)

        # Exit keywords → caller handles the break
        if user_text.strip().lower() in ["bye", "goodbye", "exit", "quit", "cya", "see you"]:
            return f"Goodbye, {self.user_name}! It was great chatting with you. Take care! 👋"

        if rule_reply is not None:
            return rule_reply

        # AI response
        if self.has_api:
            self.history.append({"role": "user", "content": user_text})
            self._trim_history()
            print_typing()
            reply = call_claude_api(
                api_key=self.api_key,
                model=self.model,
                system_prompt=self.system,
                messages=self.history,
                max_tokens=self.max_tokens,
            )
            clear_typing()
            if not reply.startswith("["):  # not an error
                self.history.append({"role": "assistant", "content": reply})
            return reply

        # Offline fallback
        return (
            "I didn't quite understand that. Could you rephrase? "
            "(Tip: set ANTHROPIC_API_KEY in .env to unlock full AI mode!)"
        )

    def run(self):
        print_banner(self.bot_name, self.has_api)
        print_bot(self.bot_name, f"Hi {self.user_name}! I'm {self.bot_name}. How can I help you today?")
        print_separator()

        while True:
            try:
                user_input = print_user_prompt(self.user_name)

                if not user_input.strip():
                    continue

                self.msg_count += 1
                response = self.get_response(user_input)
                print_bot(self.bot_name, response)
                print_separator()

                # Exit after printing goodbye
                if user_input.strip().lower() in ["bye", "goodbye", "exit", "quit", "cya", "see you"]:
                    break

            except (KeyboardInterrupt, SystemExit):
                print(f"\n\n{c('[Ctrl+C detected]', 'yellow')} {self.bot_name}: Shutting down safely. Goodbye! 👋")
                sys.exit(0)

        print(c(f"\n  Session ended. Total messages: {self.msg_count}", "grey"))
        print(c("=" * 60, "cyan"))


# ──────────────────────────────────────────────
# ENTRY POINT
# ──────────────────────────────────────────────

def main():
    config = load_config()
    session = ChatSession(config)
    session.run()

if __name__ == "__main__":
    main()
