import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";

// Simple localStorage "database"
const load = (key, def) => JSON.parse(localStorage.getItem(key)) || def;
const save = (key, val) => localStorage.setItem(key, JSON.stringify(val));

export default function MaxHuiaxApp() {
  const [users, setUsers] = useState(load("users", []));
  const [current, setCurrent] = useState(null);
  const [name, setName] = useState("");
  const [search, setSearch] = useState("");
  const [posts, setPosts] = useState(load("posts", []));
  const [text, setText] = useState("");
  const [friends, setFriends] = useState(load("friends", {}));
  const [dialogs, setDialogs] = useState(load("dialogs", {}));
  const [activeChat, setActiveChat] = useState(null);
  const [messageText, setMessageText] = useState("");

  useEffect(() => save("users", users), [users]);
  useEffect(() => save("posts", posts), [posts]);
  useEffect(() => save("friends", friends), [friends]);
  useEffect(() => save("dialogs", dialogs), [dialogs]);

  const login = () => {
    if (!name.trim()) return;
    let user = users.find(u => u.name === name);
    if (!user) {
      user = { id: Date.now(), name };
      setUsers([...users, user]);
    }
    setCurrent(user);
  };

  const addPost = () => {
    if (!text.trim()) return;
    setPosts([{ id: Date.now(), text, author: current.name }, ...posts]);
    setText("");
  };

  const addFriend = (userName) => {
    const list = friends[current.name] || [];
    if (!list.includes(userName)) {
      setFriends({ ...friends, [current.name]: [...list, userName] });
    }
  };

  const sendMessage = () => {
    if (!messageText || !activeChat) return;
    const key = [current.name, activeChat].sort().join("_");
    const chat = dialogs[key] || [];
    setDialogs({
      ...dialogs,
      [key]: [...chat, { author: current.name, text: messageText }]
    });
    setMessageText("");
  };

  if (!current) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gradient-to-b from-white to-sky-100">
        <Card className="p-6 w-80">
          <CardContent className="space-y-3">
            <h1 className="text-2xl font-bold text-sky-600 text-center">MAX HUIAX</h1>
            <Input placeholder="Enter name" value={name} onChange={(e) => setName(e.target.value)} />
            <Button onClick={login} className="bg-sky-500 text-white w-full">Enter</Button>
          </CardContent>
        </Card>
      </div>
    );
  }

  const friendList = friends[current.name] || [];
  const filteredUsers = users.filter(u => u.name.toLowerCase().includes(search.toLowerCase()) && u.name !== current.name);
  const chatKey = activeChat ? [current.name, activeChat].sort().join("_") : null;
  const messages = chatKey ? dialogs[chatKey] || [] : [];

  return (
    <div className="min-h-screen bg-gradient-to-b from-white to-sky-100">
      <header className="bg-white p-4 border-b flex justify-between">
        <h1 className="text-2xl font-bold text-sky-600">MAX HUIAX</h1>
        <div>{current.name}</div>
      </header>

      <div className="max-w-6xl mx-auto grid md:grid-cols-4 gap-4 p-4">
        {/* Left: profile + friends */}
        <Card>
          <CardContent className="p-3 space-y-2">
            <div className="font-bold text-sky-700">Friends</div>
            {friendList.map(f => (
              <div key={f} className="cursor-pointer" onClick={() => setActiveChat(f)}>{f}</div>
            ))}
          </CardContent>
        </Card>

        {/* Center: posts */}
        <div className="space-y-3">
          <Card>
            <CardContent className="p-3 space-y-2">
              <Textarea placeholder="Write a post..." value={text} onChange={(e) => setText(e.target.value)} />
              <Button onClick={addPost} className="bg-sky-500 text-white">Post</Button>
            </CardContent>
          </Card>

          {posts.map(p => (
            <Card key={p.id}><CardContent className="p-3"><b>{p.author}</b><div>{p.text}</div></CardContent></Card>
          ))}
        </div>

        {/* Search users */}
        <Card>
          <CardContent className="p-3 space-y-2">
            <div className="font-bold text-sky-700">Search people</div>
            <Input placeholder="Search..." value={search} onChange={(e) => setSearch(e.target.value)} />
            {filteredUsers.map(u => (
              <div key={u.id} className="flex justify-between">
                <span>{u.name}</span>
                <Button size="sm" onClick={() => addFriend(u.name)}>Add</Button>
              </div>
            ))}
          </CardContent>
        </Card>

        {/* Dialogs */}
        <Card>
          <CardContent className="p-3 space-y-2">
            <div className="font-bold text-sky-700">Dialog</div>
            <div className="h-40 overflow-auto bg-sky-50 p-2">
              {messages.map((m,i)=>(<div key={i}><b>{m.author}:</b> {m.text}</div>))}
            </div>
            {activeChat && (
              <div className="flex gap-2">
                <Input value={messageText} onChange={(e)=>setMessageText(e.target.value)} />
                <Button onClick={sendMessage}>Send</Button>
              </div>
            )}
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
