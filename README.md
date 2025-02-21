# Sistema-monitoramento-Aedes
function OccurrenceForm() { const [image, setImage] = useState(null); const [location, setLocation] = useState(null); const [description, setDescription] = useState("");

const handleImageUpload = async (e) => { const file = e.target.files[0]; if (!file) return; const storageRef = ref(storage, occurrences/${file.name}); await uploadBytes(storageRef, file); const url = await getDownloadURL(storageRef); setImage(url); };

const getLocation = () => { if (navigator.geolocation) { navigator.geolocation.getCurrentPosition((position) => { setLocation({ lat: position.coords.latitude, lng: position.coords.longitude, }); }); } };

const handleSubmit = async (e) => { e.preventDefault(); if (!image || !location || !description) return; await addDoc(collection(db, "occurrences"), { image, location, description, timestamp: new Date() }); alert("Ocorrência registrada com sucesso!"); };

return ( <div> <h2>Registrar Ocorrência</h2> <form onSubmit={handleSubmit}> <input type="file" onChange={handleImageUpload} accept="image/*" /> <button type="button" onClick={getLocation}>Obter Localização</button> <textarea placeholder="Descreva a ocorrência" value={description} onChange={(e) => setDescription(e.target.value)} ></textarea> <button type="submit">Enviar</button> </form> </div> ); }

export default OccurrenceForm;

