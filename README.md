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

