import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Dialog, DialogContent, DialogTrigger, DialogTitle } from "@/components/ui/dialog";

const API_URL = "https://jsonplaceholder.typicode.com/users";

export default function UserManagement() {
  const [users, setUsers] = useState([]);
  const [currentUser, setCurrentUser] = useState(null);
  const [open, setOpen] = useState(false);
  const [formData, setFormData] = useState({ id: "", name: "", email: "", department: "" });
  const [error, setError] = useState("");
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    fetch(`${API_URL}?_limit=5&_page=${page}`)
      .then(response => {
        if (!response.ok) throw new Error("Failed to fetch users");
        return response.json();
      })
      .then(data => setUsers(prev => [...prev, ...data.map(user => ({ id: user.id, name: user.name, email: user.email, department: "N/A" }))]))
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [page]);

  useEffect(() => {
    if (currentUser) {
      setFormData(currentUser);
    } else {
      setFormData({ id: "", name: "", email: "", department: "" });
    }
  }, [currentUser]);

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData({ ...formData, [name]: value });
  };

  const handleSubmit = () => {
    if (!formData.name || !formData.email) {
      setError("Name and Email are required");
      return;
    }

    const method = formData.id ? "PUT" : "POST";
    const url = formData.id ? `${API_URL}/${formData.id}` : API_URL;

    fetch(url, {
      method,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(formData)
    })
      .then(response => {
        if (!response.ok) throw new Error(`Failed to ${formData.id ? "update" : "add"} user`);
        return response.json();
      })
      .then(user => {
        setUsers(formData.id ? users.map(u => (u.id === user.id ? user : u)) : [...users, { ...user, id: users.length + 1 }]);
        setOpen(false);
        setCurrentUser(null);
        setError("");
      })
      .catch(err => setError(err.message));
  };

  const handleDelete = (id) => {
    fetch(`${API_URL}/${id}`, { method: "DELETE" })
      .then(response => {
        if (!response.ok) throw new Error("Failed to delete user");
        setUsers(users.filter(user => user.id !== id));
      })
      .catch(err => setError(err.message));
  };

  return (
    <div className="p-6 max-w-xl mx-auto">
      <h1 className="text-2xl font-bold mb-4 text-center">User Management</h1>
      {error && <p className="text-red-500 text-center">Error: {error}</p>}
      <Button onClick={() => { setCurrentUser(null); setOpen(true); }}>Add User</Button>
      <div className="mt-4">
        {users.map(user => (
          <Card key={user.id} className="mb-2 p-4 flex justify-between items-center">
            <CardContent>
              <p><strong>ID:</strong> {user.id}</p>
              <p><strong>Name:</strong> {user.name}</p>
              <p><strong>Email:</strong> {user.email}</p>
              <p><strong>Department:</strong> {user.department}</p>
            </CardContent>
            <div>
              <Button onClick={() => { setCurrentUser(user); setOpen(true); }}>Edit</Button>
              <Button className="ml-2" variant="destructive" onClick={() => handleDelete(user.id)}>Delete</Button>
            </div>
          </Card>
        ))}
      </div>
      {loading && <p className="text-center">Loading more users...</p>}
      <Button onClick={() => setPage(prev => prev + 1)} disabled={loading} className="mt-4 w-full">Load More</Button>
      <Dialog open={open} onOpenChange={setOpen}>
        <DialogContent>
          <DialogTitle>{currentUser ? "Edit User" : "Add User"}</DialogTitle>
          <Input name="name" value={formData.name} onChange={handleInputChange} placeholder="Name" className="mt-2" required />
          <Input name="email" value={formData.email} onChange={handleInputChange} placeholder="Email" className="mt-2" required />
          <Input name="department" value={formData.department} onChange={handleInputChange} placeholder="Department" className="mt-2" />
          <Button className="mt-4 w-full" onClick={handleSubmit}>{currentUser ? "Update" : "Add"} User</Button>
        </DialogContent>
      </Dialog>
    </div>
  );
}
