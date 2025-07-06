# MERN-assn
import express from 'express';
import mongoose from 'mongoose';
import dotenv from 'dotenv';
import cors from 'cors';

import postRoutes from './routes/posts.js';
import userRoutes from './routes/users.js';

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

app.use('/api/posts', postRoutes);
app.use('/api/users', userRoutes);

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true })
  .then(() => app.listen(5000, () => console.log("Server started on 5000")))
  .catch(err => console.error(err));
import mongoose from 'mongoose';

const postSchema = mongoose.Schema({
  title: String,
  content: String,
  image: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  comments: [{ text: String, date: Date }]
});

export default mongoose.model('Post', postSchema);
import mongoose from 'mongoose';

const userSchema = mongoose.Schema({
  username: String,
  password: String,
});

export default mongoose.model('User', userSchema);
import express from 'express';
import { getPosts, createPost } from '../controllers/posts.js';
const router = express.Router();

router.get('/', getPosts);
router.post('/', createPost);

export default router;
import Post from '../models/Post.js';

export const getPosts = async (req, res) => {
  const posts = await Post.find().populate('author');
  res.json(posts);
};

export const createPost = async (req, res) => {
  const post = new Post(req.body);
  await post.save();
  res.status(201).json(post);
};
import { createContext, useState, useEffect } from 'react';

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const login = (data) => setUser(data);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
import { useState } from 'react';
import { api } from '../services/api';

export default function PostForm() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    await api.post('/posts', { title, content });
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input value={title} onChange={(e) => setTitle(e.target.value)} placeholder="Title" />
      <textarea value={content} onChange={(e) => setContent(e.target.value)} placeholder="Content" />
      <button type="submit">Submit</button>
    </form>
  );
}
