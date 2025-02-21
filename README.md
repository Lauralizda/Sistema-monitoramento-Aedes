# Sistema-monitoramento-Aedes
function OccurrenceForm() { const [image, setImage] = useState(null); const [location, setLocation] = useState(null); const [description, setDescription] = useState("");

const handleImageUpload = async (e) => { const file = e.target.files[0]; if (!file) return; const storageRef = ref(storage, occurrences/${file.name}); await uploadBytes(storageRef, file); const url = await getDownloadURL(storageRef); setImage(url); };

const getLocation = () => { if (navigator.geolocation) { navigator.geolocation.getCurrentPosition((position) => { setLocation({ lat: position.coords.latitude, lng: position.coords.longitude, }); }); } };

const handleSubmit = async (e) => { e.preventDefault(); if (!image || !location || !description) return; await addDoc(collection(db, "occurrences"), { image, location, description, timestamp: new Date() }); alert("Ocorrência registrada com sucesso!"); };

return ( <div> <h2>Registrar Ocorrência</h2> <form onSubmit={handleSubmit}> <input type="file" onChange={handleImageUpload} accept="image/*" /> <button type="button" onClick={getLocation}>Obter Localização</button> <textarea placeholder="Descreva a ocorrência" value={description} onChange={(e) => setDescription(e.target.value)} ></textarea> <button type="submit">Enviar</button> </form> </div> ); }

export default OccurrenceForm;

// src/App.js - Estrutura inicial do sistema import React from 'react'; import { BrowserRouter as Router, Routes, Route } from 'react-router-dom'; import Home from './pages/Home'; import Report from './pages/Report'; import Dashboard from './pages/Dashboard'; import Navbar from './components/Navbar'; import OccurrenceForm from './pages/OccurrenceForm'; import OccurrenceList from './pages/OccurrenceList';

function App() { return ( <Router> <Navbar /> <Routes> <Route path='/' element={<Home />} /> <Route path='/report' element={<Report />} /> <Route path='/dashboard' element={<Dashboard />} /> <Route path='/occurrence' element={<OccurrenceForm />} /> <Route path='/occurrences' element={<OccurrenceList />} /> </Routes> </Router> ); }

export default App;

// src/pages/OccurrenceList.js - Listagem e filtragem de ocorrências import React, { useState, useEffect } from 'react'; import { getFirestore, collection, query, where, getDocs } from "firebase/firestore"; import { app } from "../firebaseConfig";

const db = getFirestore(app);

function OccurrenceList() { const [occurrences, setOccurrences] = useState([]); const [filter, setFilter] = useState("");

useEffect(() => { const fetchOccurrences = async () => { let q = collection(db, "occurrences"); if (filter) { q = query(q, where("description", ">=", filter)); } const querySnapshot = await getDocs(q); const data = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })); setOccurrences(data); }; fetchOccurrences(); }, [filter]);

return ( <div> <h2>Lista de Ocorrências</h2> <input type="text" placeholder="Filtrar por descrição..." value={filter} onChange={(e) => setFilter(e.target.value)} /> <ul> {occurrences.map((occurrence) => ( <li key={occurrence.id}> <img src={occurrence.image} alt="Ocorrência" width="100" /> <p>{occurrence.description}</p> <p>Localização: {occurrence.location.lat}, {occurrence.location.lng}</p> </li> ))} </ul> </div> ); }

export default OccurrenceList;
// src/App.js - Estrutura inicial do sistema import React from 'react'; import { BrowserRouter as Router, Routes, Route } from 'react-router-dom'; import Home from './pages/Home'; import Report from './pages/Report'; import Dashboard from './pages/Dashboard'; import Navbar from './components/Navbar'; import OccurrenceForm from './pages/OccurrenceForm'; import OccurrenceList from './pages/OccurrenceList';

function App() { return ( <Router> <Navbar /> <Routes> <Route path='/' element={<Home />} /> <Route path='/report' element={<Report />} /> <Route path='/dashboard' element={<Dashboard />} /> <Route path='/occurrence' element={<OccurrenceForm />} /> <Route path='/occurrences' element={<OccurrenceList />} /> </Routes> </Router> ); }

export default App;

// src/pages/OccurrenceList.js - Listagem e filtragem de ocorrências import React, { useState, useEffect } from 'react'; import { getFirestore, collection, query, where, getDocs, orderBy } from "firebase/firestore"; import { app } from "../firebaseConfig";

const db = getFirestore(app);

function OccurrenceList() { const [occurrences, setOccurrences] = useState([]); const [filter, setFilter] = useState(""); const [dateFilter, setDateFilter] = useState(""); const [locationFilter, setLocationFilter] = useState("");

useEffect(() => { const fetchOccurrences = async () => { let q = collection(db, "occurrences"); if (filter) { q = query(q, where("description", ">=", filter)); } if (dateFilter) { q = query(q, orderBy("timestamp"), where("timestamp", ">=", new Date(dateFilter))); } if (locationFilter) { q = query(q, where("location", "==", locationFilter)); } const querySnapshot = await getDocs(q); const data = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })); setOccurrences(data); }; fetchOccurrences(); }, [filter, dateFilter, locationFilter]);

return ( <div> <h2>Lista de Ocorrências</h2> <input type="text" placeholder="Filtrar por descrição..." value={filter} onChange={(e) => setFilter(e.target.value)} /> <input type="date" value={dateFilter} onChange={(e) => setDateFilter(e.target.value)} /> <input type="text" placeholder="Filtrar por localização..." value={locationFilter} onChange={(e) => setLocationFilter(e.target.value)} /> <ul> {occurrences.map((occurrence) => ( <li key={occurrence.id}> <img src={occurrence.image} alt="Ocorrência" width="100" /> <p>{occurrence.description}</p> <p>Data: {new Date(occurrence.timestamp?.toDate()).toLocaleDateString()}</p> <p>Localização: {occurrence.location.lat}, {occurrence.location.lng}</p> </li> ))} </ul> </div> ); }

export default OccurrenceList;
// src/App.js - Estrutura inicial do sistema import React, { useEffect } from 'react'; import { BrowserRouter as Router, Routes, Route } from 'react-router-dom'; import Home from './pages/Home'; import Report from './pages/Report'; import Dashboard from './pages/Dashboard'; import Navbar from './components/Navbar'; import OccurrenceForm from './pages/OccurrenceForm'; import OccurrenceList from './pages/OccurrenceList'; import { getMessaging, onMessage } from "firebase/messaging"; import { app } from "./firebaseConfig";
function App() { useEffect(() => { const messaging = getMessaging(app); onMessage(messaging, (payload) => { alert(Nova ocorrência registrada: ${payload.notification.title}); }); }, []);

return ( <Router> <Navbar /> <Routes> <Route path='/' element={<Home />} /> <Route path='/report' element={<Report />} /> <Route path='/dashboard' element={<Dashboard />} /> <Route path='/occurrence' element={<OccurrenceForm />} /> <Route path='/occurrences' element={<OccurrenceList />} /> </Routes> </Router> ); }

export default App;

// Firebase Cloud Messaging Configuração import { getToken } from "firebase/messaging";

export const requestNotificationPermission = async () => { const messaging = getMessaging(app); try { const permission = await Notification.requestPermission(); if (permission === "granted") { const token = await getToken(messaging, { vapidKey: "SUA_VAPID_KEY" }); console.log("Token de notificação registrado:", token); } } catch (error) { console.error("Erro ao obter permissão para notificações", error); } };
// src/App.js - Estrutura inicial do sistema import React, { useEffect, useState } from 'react'; import { BrowserRouter as Router, Routes, Route } from 'react-router-dom'; import Home from './pages/Home'; import Report from './pages/Report'; import Dashboard from './pages/Dashboard'; import Navbar from './components/Navbar'; import OccurrenceForm from './pages/OccurrenceForm'; import OccurrenceList from './pages/OccurrenceList'; import Login from './pages/Login'; import { getMessaging, onMessage } from "firebase/messaging"; import { app, auth } from "./firebaseConfig"; import { onAuthStateChanged } from "firebase/auth";

function App() { const [user, setUser] = useState(null);

useEffect(() => { const messaging = getMessaging(app); onMessage(messaging, (payload) => { alert(Nova ocorrência registrada: ${payload.notification.title}); }); }, []);

useEffect(() => { onAuthStateChanged(auth, (user) => { setUser(user); }); }, []);

return ( <Router> <Navbar user={user} /> <Routes> <Route path='/' element={user ? <Home /> : <Login />} /> <Route path='/report' element={user ? <Report /> : <Login />} /> <Route path='/dashboard' element={user ? <Dashboard /> : <Login />} /> <Route path='/occurrence' element={user ? <OccurrenceForm /> : <Login />} /> <Route path='/occurrences' element={user ? <OccurrenceList /> : <Login />} /> </Routes> </Router> ); }

export default App;

// src/pages/Login.js - Página de Autenticação import React from 'react'; import { signInWithPopup, GoogleAuthProvider } from "firebase/auth"; import { auth } from "../firebaseConfig";

function Login() { const handleLogin = async () => { const provider = new GoogleAuthProvider(); try { await signInWithPopup(auth, provider); } catch (error) { console.error("Erro ao fazer login", error); } };

return ( <div> <h2>Login</h2> <button onClick={handleLogin}>Entrar com Google</button> </div> ); }

export default Login;

// src/firebaseConfig.js - Configuração do Firebase import { initializeApp } from "firebase/app"; import { getAuth } from "firebase/auth"; import { getFirestore } from "firebase/firestore"; import { getMessaging } from "firebase/messaging";

const firebaseConfig = { apiKey: "SUA_API_KEY", authDomain: "SEU_AUTH_DOMAIN", projectId: "SEU_PROJECT_ID", storageBucket: "SEU_STORAGE_BUCKET", messagingSenderId: "SEU_MESSAGING_SENDER_ID", appId: "SEU_APP_ID" };

export const app = initializeApp(firebaseConfig); export const auth = getAuth(app); export const db = getFirestore(app); export const messaging = getMessaging(app);






