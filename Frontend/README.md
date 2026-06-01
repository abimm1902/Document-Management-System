
import { useState, useEffect, useRef, useCallback } from "react";

const API = "http://localhost:5000/api";

// ─── QUANTIC DESIGN TOKENS ───────────────────
const C = {
  bgDeep: "#0d1117", bgMain: "#161b22", bgCard: "#1c2128", bgHover: "#21262d",
  bgInput: "#0d1117", bgSidebar: "#0d1117", bgSideHover: "#1c2128",
  accentBlue: "#2f81f7", accentBlueBg: "#1a3a5c", accentGreen: "#2ea043",
  accentGreenBg: "#1a3524", accentYellow: "#d29922", accentYellowBg: "#3d2e00",
  accentOrange: "#db6d28", accentOrangeBg: "#3d2200", accentRed: "#f85149",
  accentPurple: "#a371f7",
  textPrimary: "#e6edf3", textSecondary: "#8b949e", textTertiary: "#484f58",
  textLink: "#58a6ff", textWhite: "#ffffff",
  border: "#30363d", borderLight: "#21262d", borderFocus: "#2f81f7",
  statusNew: "#2ea043", statusProgress: "#d29922", statusWaiting: "#db6d28",
  statusDraft: "#484f58", statusPublished: "#2ea043", statusReview: "#d29922",
};

const T = {
  font: `'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif`,
  fontMono: `'SF Mono', 'Cascadia Code', 'Consolas', monospace`,
  size: { xs: 10, sm: 11, md: 13, base: 14, lg: 16, xl: 20, xxl: 24 },
  weight: { normal: 400, medium: 500, semi: 600, bold: 700 },
  radius: { sm: 4, md: 6, lg: 8, xl: 12, pill: 9999 },
};

// ─── BADGES ──────────────────────────────────
const Badge = ({ status }) => {
  const map = {
    published: { bg: C.accentGreenBg, color: C.accentGreen, border: C.accentGreen, label: "Published" },
    draft: { bg: C.bgHover, color: C.textSecondary, border: C.textTertiary, label: "Draft" },
    review: { bg: C.accentYellowBg, color: C.accentYellow, border: C.accentYellow, label: "In Review" },
    new: { bg: C.accentGreenBg, color: C.accentGreen, border: C.accentGreen, label: "New" },
    waiting: { bg: C.accentOrangeBg, color: C.accentOrange, border: C.accentOrange, label: "Waiting On Client" },
  };
  const s = map[status] || map.draft;
  return (
    <span style={{ display: "inline-block", padding: "2px 10px", borderRadius: T.radius.sm,
      fontSize: T.size.sm, fontWeight: T.weight.semi, background: s.bg, color: s.color,
      border: `1px solid ${s.border}30` }}>{s.label}</span>
  );
};

const TypeBadge = ({ type }) => {
  const colors = { PDF: C.accentRed, DOCX: C.accentBlue, XLSX: C.accentGreen };
  const c = colors[type] || C.textSecondary;
  return (
    <span style={{ fontSize: T.size.xs, fontWeight: T.weight.bold, padding: "1px 6px",
      borderRadius: T.radius.sm, background: `${c}20`, color: c, letterSpacing: 0.3 }}>{type}</span>
  );
};

// ─── TOAST NOTIFICATION ──────────────────────
const Toast = ({ msg, type, onClose }) => {
  useEffect(() => { const t = setTimeout(onClose, 3000); return () => clearTimeout(t); }, [onClose]);
  const bg = type === "error" ? C.accentRed : type === "warn" ? C.accentYellow : C.accentGreen;
  return (
    <div style={{ position: "fixed", bottom: 24, right: 24, zIndex: 9999,
      background: C.bgCard, border: `1px solid ${bg}`, borderRadius: T.radius.lg,
      padding: "12px 18px", color: C.textPrimary, fontSize: T.size.base,
      boxShadow: "0 8px 24px rgba(0,0,0,.5)", display: "flex", alignItems: "center", gap: 10 }}>
      <span style={{ color: bg, fontSize: 18 }}>{type === "error" ? "✕" : "✓"}</span>
      {msg}
      <span onClick={onClose} style={{ marginLeft: 8, cursor: "pointer", color: C.textTertiary }}>✕</span>
    </div>
  );
};

// ─── MODAL WRAPPER ───────────────────────────
const Modal = ({ title, onClose, children }) => (
  <div style={{ position: "fixed", inset: 0, zIndex: 1000, background: "rgba(0,0,0,.7)",
    display: "flex", alignItems: "center", justifyContent: "center" }}
    onClick={e => e.target === e.currentTarget && onClose()}>
    <div style={{ background: C.bgCard, border: `1px solid ${C.border}`, borderRadius: T.radius.xl,
      padding: 28, minWidth: 480, maxWidth: 640, width: "90%", boxShadow: "0 24px 64px rgba(0,0,0,.7)" }}>
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
        <div style={{ fontSize: T.size.lg, fontWeight: T.weight.bold, color: C.textPrimary }}>{title}</div>
        <button onClick={onClose} style={{ background: "none", border: "none", color: C.textSecondary,
          cursor: "pointer", fontSize: 20, padding: "0 4px" }}>✕</button>
      </div>
      {children}
    </div>
  </div>
);

// ─── FORM INPUT HELPER ───────────────────────
const Field = ({ label, children }) => (
  <div style={{ marginBottom: 16 }}>
    <label style={{ display: "block", fontSize: T.size.sm, fontWeight: T.weight.semi,
      color: C.textSecondary, marginBottom: 6, textTransform: "uppercase", letterSpacing: 0.5 }}>{label}</label>
    {children}
  </div>
);
const Input = (props) => (
  <input {...props} style={{ width: "100%", boxSizing: "border-box", padding: "9px 12px",
    borderRadius: T.radius.md, border: `1px solid ${C.border}`, background: C.bgInput,
    color: C.textPrimary, fontSize: T.size.base, fontFamily: T.font, outline: "none",
    ...(props.style || {}) }} />
);
const Select = ({ value, onChange, children, style }) => (
  <select value={value} onChange={onChange} style={{ width: "100%", boxSizing: "border-box",
    padding: "9px 12px", borderRadius: T.radius.md, border: `1px solid ${C.border}`,
    background: C.bgInput, color: C.textPrimary, fontSize: T.size.base, fontFamily: T.font,
    outline: "none", ...(style || {}) }}>
    {children}
  </select>
);
const Btn = ({ onClick, children, variant = "primary", style = {}, disabled }) => {
  const bg = variant === "primary" ? C.accentBlue : variant === "danger" ? C.accentRed : C.bgCard;
  const border = variant === "secondary" ? `1px solid ${C.border}` : "none";
  return (
    <button onClick={onClick} disabled={disabled} style={{
      padding: "9px 18px", borderRadius: T.radius.md, border, background: bg,
      color: variant === "secondary" ? C.textSecondary : "#fff",
      fontSize: T.size.base, fontWeight: T.weight.semi, cursor: disabled ? "not-allowed" : "pointer",
      fontFamily: T.font, opacity: disabled ? 0.6 : 1, ...style }}>
      {children}
    </button>
  );
};

// ─── SIDEBAR ─────────────────────────────────
const Sidebar = ({ activePanel, setActivePanel, stats }) => {
  const navItems = [
    { icon: "⊞", label: "Dashboard" }, { icon: "📊", label: "Reports" },
    { icon: "📋", label: "Leads" }, { icon: "👥", label: "Contacts" },
    { icon: "🎫", label: "Tickets" }, { icon: "🏢", label: "Accounts" },
    { icon: "👤", label: "Employees" }, { icon: "🤝", label: "Partners" },
  ];
  const docNav = [
    { key: "documents", icon: "📄", label: "Documents", count: stats.total || 0 },
    { key: "faq", icon: "❓", label: "FAQs", count: stats.faqCount || 0 },
    { key: "search", icon: "🔍", label: "Content Search" },
  ];
  return (
    <div style={{ width: 240, background: C.bgSidebar, borderRight: `1px solid ${C.border}`,
      display: "flex", flexDirection: "column", flexShrink: 0, overflow: "hidden" }}>
      <div style={{ padding: "18px 20px 14px", display: "flex", alignItems: "center", gap: 10,
        borderBottom: `1px solid ${C.border}` }}>
        <div style={{ width: 32, height: 32, borderRadius: T.radius.lg, background: C.accentBlue,
          display: "flex", alignItems: "center", justifyContent: "center", fontSize: 14, color: "#fff",
          fontWeight: T.weight.bold }}>Q</div>
        <span style={{ fontSize: T.size.lg, fontWeight: T.weight.bold, color: C.textPrimary,
          letterSpacing: 1.5, textTransform: "uppercase" }}>Quantic<span style={{ fontSize: 8,
          verticalAlign: "super", marginLeft: 1 }}>®</span></span>
      </div>
      <div style={{ flex: 1, overflowY: "auto", padding: "8px 0" }}>
        {navItems.map(item => (
          <div key={item.label} style={{ display: "flex", alignItems: "center", gap: 12,
            padding: "9px 20px", cursor: "pointer", color: C.textSecondary, fontSize: T.size.base }}>
            <span style={{ fontSize: 16, width: 20, textAlign: "center" }}>{item.icon}</span>
            {item.label}
          </div>
        ))}
        <div style={{ margin: "8px 12px", borderTop: `1px solid ${C.border}`, paddingTop: 8 }}>
          <div style={{ fontSize: T.size.xs, fontWeight: T.weight.bold, textTransform: "uppercase",
            letterSpacing: 1, color: C.textTertiary, padding: "6px 8px" }}>Document hub</div>
          {docNav.map(item => (
            <div key={item.key} onClick={() => setActivePanel(item.key)} style={{
              display: "flex", alignItems: "center", gap: 12, padding: "9px 12px",
              cursor: "pointer", borderRadius: T.radius.md, marginBottom: 2,
              color: activePanel === item.key ? C.textWhite : C.textSecondary,
              background: activePanel === item.key ? C.accentBlueBg : "transparent",
              borderLeft: activePanel === item.key ? `3px solid ${C.accentBlue}` : "3px solid transparent",
              fontSize: T.size.base, fontWeight: activePanel === item.key ? T.weight.semi : T.weight.normal,
            }}>
              <span style={{ fontSize: 14 }}>{item.icon}</span>
              {item.label}
              {item.count !== undefined && (
                <span style={{ marginLeft: "auto", fontSize: T.size.xs, background: C.bgHover,
                  color: C.textSecondary, padding: "1px 8px", borderRadius: T.radius.pill,
                  fontWeight: T.weight.semi }}>{item.count}</span>
              )}
            </div>
          ))}
        </div>
      </div>
      <div style={{ padding: "12px 20px", borderTop: `1px solid ${C.border}`,
        fontSize: T.size.sm, color: C.textTertiary }}>THEME</div>
    </div>
  );
};

// ─── UPLOAD MODAL ────────────────────────────
const UploadModal = ({ onClose, onSuccess, collections }) => {
  const [file, setFile] = useState(null);
  const [form, setForm] = useState({ title: "", collection: "", author: "", status: "draft", pages: "" });
  const [loading, setLoading] = useState(false);
  const [drag, setDrag] = useState(false);
  const fileRef = useRef();

  const handleFile = (f) => {
    if (!f) return;
    setFile(f);
    if (!form.title) setForm(p => ({ ...p, title: f.name.replace(/\.[^.]+$/, "") }));
  };

  const submit = async () => {
    if (!file) return;
    setLoading(true);
    const fd = new FormData();
    fd.append("file", file);
    Object.entries(form).forEach(([k, v]) => v && fd.append(k, v));
    try {
      const res = await fetch(`${API}/documents/upload`, { method: "POST", body: fd });
      if (!res.ok) throw new Error((await res.json()).error);
      const doc = await res.json();
      onSuccess(doc);
      onClose();
    } catch (e) {
      alert("Upload failed: " + e.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <Modal title="Upload Document" onClose={onClose}>
      <div onDragOver={e => { e.preventDefault(); setDrag(true); }}
        onDragLeave={() => setDrag(false)}
        onDrop={e => { e.preventDefault(); setDrag(false); handleFile(e.dataTransfer.files[0]); }}
        onClick={() => !file && fileRef.current.click()}
        style={{ border: `2px dashed ${drag ? C.accentBlue : C.border}`, borderRadius: T.radius.lg,
          padding: "28px 20px", textAlign: "center", marginBottom: 20, cursor: "pointer",
          background: drag ? C.accentBlueBg : C.bgDeep, transition: "all 0.2s" }}>
        <input ref={fileRef} type="file" style={{ display: "none" }}
          accept=".pdf,.docx,.xlsx,.png,.jpg,.jpeg"
          onChange={e => handleFile(e.target.files[0])} />
        {file ? (
          <div>
            <div style={{ fontSize: 32, marginBottom: 8 }}>📄</div>
            <div style={{ color: C.textPrimary, fontWeight: T.weight.semi }}>{file.name}</div>
            <div style={{ color: C.textSecondary, fontSize: T.size.sm, marginTop: 4 }}>
              {(file.size / 1024 / 1024).toFixed(2)} MB
            </div>
            <button onClick={e => { e.stopPropagation(); setFile(null); }}
              style={{ marginTop: 8, background: "none", border: "none", color: C.accentRed, cursor: "pointer", fontSize: T.size.sm }}>
              Remove
            </button>
          </div>
        ) : (
          <div>
            <div style={{ fontSize: 32, marginBottom: 8 }}>☁️</div>
            <div style={{ color: C.textSecondary, fontSize: T.size.base }}>
              Drag & drop or <span style={{ color: C.accentBlue }}>browse</span>
            </div>
            <div style={{ color: C.textTertiary, fontSize: T.size.sm, marginTop: 4 }}>
              PDF, DOCX, XLSX, PNG, JPG · Max 50MB
            </div>
          </div>
        )}
      </div>
      <Field label="Title">
        <Input value={form.title} onChange={e => setForm(p => ({ ...p, title: e.target.value }))} placeholder="Document title" />
      </Field>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
        <Field label="Collection">
          <Input value={form.collection} onChange={e => setForm(p => ({ ...p, collection: e.target.value }))}
            placeholder="e.g. Billing" list="collist" />
          <datalist id="collist">{collections.map(c => <option key={c} value={c} />)}</datalist>
        </Field>
        <Field label="Author">
          <Input value={form.author} onChange={e => setForm(p => ({ ...p, author: e.target.value }))} placeholder="Author name" />
        </Field>
        <Field label="Status">
          <Select value={form.status} onChange={e => setForm(p => ({ ...p, status: e.target.value }))}>
            <option value="draft">Draft</option>
            <option value="review">In Review</option>
            <option value="published">Published</option>
          </Select>
        </Field>
        <Field label="Pages">
          <Input type="number" value={form.pages} onChange={e => setForm(p => ({ ...p, pages: e.target.value }))} placeholder="Number of pages" />
        </Field>
      </div>
      <div style={{ display: "flex", justifyContent: "flex-end", gap: 10, marginTop: 8 }}>
        <Btn variant="secondary" onClick={onClose}>Cancel</Btn>
        <Btn onClick={submit} disabled={!file || loading}>{loading ? "Uploading..." : "Upload Document"}</Btn>
      </div>
    </Modal>
  );
};

// ─── EDIT DOCUMENT MODAL ─────────────────────
const EditDocModal = ({ doc, onClose, onSuccess, collections }) => {
  const [form, setForm] = useState({ title: doc.title, collection: doc.collection,
    author: doc.author, status: doc.status, pages: doc.pages, type: doc.type });
  const [loading, setLoading] = useState(false);

  const submit = async () => {
    setLoading(true);
    try {
      const res = await fetch(`${API}/documents/${doc.id}`, {
        method: "PUT", headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form),
      });
      if (!res.ok) throw new Error("Update failed");
      onSuccess(await res.json());
      onClose();
    } catch (e) {
      alert(e.message);
    } finally { setLoading(false); }
  };

  return (
    <Modal title={`Edit — ${doc.id}`} onClose={onClose}>
      <Field label="Title"><Input value={form.title} onChange={e => setForm(p => ({ ...p, title: e.target.value }))} /></Field>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
        <Field label="Collection">
          <Input value={form.collection} onChange={e => setForm(p => ({ ...p, collection: e.target.value }))} list="collist2" />
          <datalist id="collist2">{collections.map(c => <option key={c} value={c} />)}</datalist>
        </Field>
        <Field label="Author"><Input value={form.author} onChange={e => setForm(p => ({ ...p, author: e.target.value }))} /></Field>
        <Field label="Status">
          <Select value={form.status} onChange={e => setForm(p => ({ ...p, status: e.target.value }))}>
            <option value="draft">Draft</option><option value="review">In Review</option><option value="published">Published</option>
          </Select>
        </Field>
        <Field label="Type">
          <Select value={form.type} onChange={e => setForm(p => ({ ...p, type: e.target.value }))}>
            <option>PDF</option><option>DOCX</option><option>XLSX</option>
          </Select>
        </Field>
      </div>
      <div style={{ display: "flex", justifyContent: "flex-end", gap: 10, marginTop: 8 }}>
        <Btn variant="secondary" onClick={onClose}>Cancel</Btn>
        <Btn onClick={submit} disabled={loading}>{loading ? "Saving..." : "Save Changes"}</Btn>
      </div>
    </Modal>
  );
};

// ─── DOCUMENTS PANEL ─────────────────────────
const DocumentsPanel = ({ onToast }) => {
  const [tab, setTab] = useState("All");
  const [viewMode, setViewMode] = useState("Table");
  const [docs, setDocs] = useState([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [perPage, setPerPage] = useState(10);
  const [search, setSearch] = useState("");
  const [searchInput, setSearchInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [collections, setCollections] = useState([]);
  const [filterCollection, setFilterCollection] = useState("All");
  const [filterType, setFilterType] = useState("All");
  const [sort, setSort] = useState("updated_desc");
  const [showUpload, setShowUpload] = useState(false);
  const [editDoc, setEditDoc] = useState(null);
  const [deleteConfirm, setDeleteConfirm] = useState(null);
  const debounceRef = useRef();

  const tabToStatus = { All: undefined, Published: "published", Drafts: "draft", "In Review": "review" };

  const load = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ page, per_page: perPage, sort });
      const status = tabToStatus[tab];
      if (status) params.set("status", status);
      if (filterCollection !== "All") params.set("collection", filterCollection);
      if (filterType !== "All") params.set("type", filterType);
      if (search) params.set("search", search);
      const res = await fetch(`${API}/documents?${params}`);
      const data = await res.json();
      setDocs(data.data);
      setTotal(data.total);
    } catch { onToast("Failed to load documents", "error"); }
    finally { setLoading(false); }
  }, [page, perPage, sort, tab, filterCollection, filterType, search]);

  useEffect(() => { load(); }, [load]);

  useEffect(() => {
    fetch(`${API}/collections`).then(r => r.json()).then(setCollections).catch(() => {});
  }, []);

  const handleSearchInput = (v) => {
    setSearchInput(v);
    clearTimeout(debounceRef.current);
    debounceRef.current = setTimeout(() => { setSearch(v); setPage(1); }, 300);
  };

  const deleteDoc = async (id) => {
    try {
      await fetch(`${API}/documents/${id}`, { method: "DELETE" });
      onToast("Document deleted", "success");
      setDeleteConfirm(null);
      load();
    } catch { onToast("Delete failed", "error"); }
  };

  const tabs = ["All", "Published", "Drafts", "In Review"];
  const totalPages = Math.ceil(total / perPage);

  return (
    <div style={{ display: "flex", flexDirection: "column", height: "100%" }}>
      {showUpload && <UploadModal onClose={() => setShowUpload(false)} onSuccess={() => { load(); onToast("Document uploaded!", "success"); }} collections={collections} />}
      {editDoc && <EditDocModal doc={editDoc} onClose={() => setEditDoc(null)} onSuccess={() => { setEditDoc(null); load(); onToast("Document updated!", "success"); }} collections={collections} />}
      {deleteConfirm && (
        <Modal title="Delete Document" onClose={() => setDeleteConfirm(null)}>
          <p style={{ color: C.textSecondary, marginBottom: 20 }}>
            Are you sure you want to delete <strong style={{ color: C.textPrimary }}>{deleteConfirm.title}</strong>? This cannot be undone.
          </p>
          <div style={{ display: "flex", justifyContent: "flex-end", gap: 10 }}>
            <Btn variant="secondary" onClick={() => setDeleteConfirm(null)}>Cancel</Btn>
            <Btn variant="danger" onClick={() => deleteDoc(deleteConfirm.id)}>Delete</Btn>
          </div>
        </Modal>
      )}

      {/* Header */}
      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between",
        padding: "14px 24px", borderBottom: `1px solid ${C.border}` }}>
        <div style={{ fontSize: T.size.xl, fontWeight: T.weight.bold, color: C.textPrimary }}>Documents
          <span style={{ marginLeft: 10, fontSize: T.size.base, fontWeight: T.weight.normal, color: C.textSecondary }}>({total})</span>
        </div>
        <button onClick={() => setShowUpload(true)} style={{ padding: "8px 18px", borderRadius: T.radius.md,
          border: "none", background: C.accentBlue, color: "#fff", fontSize: T.size.base,
          fontWeight: T.weight.semi, cursor: "pointer", fontFamily: T.font, display: "flex", alignItems: "center", gap: 6 }}>
          + Upload Document
        </button>
      </div>

      {/* Tabs */}
      <div style={{ display: "flex", borderBottom: `1px solid ${C.border}`, padding: "0 24px" }}>
        {tabs.map(t => (
          <div key={t} onClick={() => { setTab(t); setPage(1); }} style={{
            padding: "10px 16px", fontSize: T.size.md, cursor: "pointer", marginBottom: -1,
            color: tab === t ? C.textWhite : C.textSecondary,
            borderBottom: `2px solid ${tab === t ? C.accentBlue : "transparent"}`,
            fontWeight: tab === t ? T.weight.semi : T.weight.normal,
          }}>{t}</div>
        ))}
      </div>

      {/* Search + View Toggle */}
      <div style={{ padding: "12px 24px", display: "flex", alignItems: "center", gap: 12 }}>
        <div style={{ flex: 1, display: "flex", alignItems: "center", gap: 8, padding: "8px 14px",
          borderRadius: T.radius.md, border: `1px solid ${C.border}`, background: C.bgInput }}>
          <span style={{ color: C.textTertiary }}>🔍</span>
          <input value={searchInput} onChange={e => handleSearchInput(e.target.value)}
            placeholder="Search documents..." style={{ border: "none", outline: "none",
            background: "transparent", flex: 1, fontSize: T.size.base, fontFamily: T.font, color: C.textPrimary }} />
          {searchInput && <span onClick={() => handleSearchInput("")} style={{ cursor: "pointer", color: C.textTertiary }}>✕</span>}
        </div>
        <div style={{ display: "flex", borderRadius: T.radius.md, overflow: "hidden", border: `1px solid ${C.border}` }}>
          {["Table", "Grid"].map(m => (
            <button key={m} onClick={() => setViewMode(m)} style={{ padding: "7px 16px", border: "none",
              fontSize: T.size.md, fontWeight: T.weight.semi, cursor: "pointer", fontFamily: T.font,
              background: viewMode === m ? C.accentBlue : C.bgCard, color: viewMode === m ? "#fff" : C.textSecondary }}>
              {m === "Table" ? "≡ " : "⊞ "}{m}
            </button>
          ))}
        </div>
      </div>

      {/* Filters */}
      <div style={{ padding: "0 24px 12px", display: "flex", gap: 8, flexWrap: "wrap", alignItems: "center" }}>
        <select value={filterCollection} onChange={e => { setFilterCollection(e.target.value); setPage(1); }}
          style={{ padding: "5px 10px", borderRadius: T.radius.md, border: `1px solid ${C.border}`,
            background: C.bgCard, fontSize: T.size.md, color: filterCollection !== "All" ? C.accentBlue : C.textSecondary,
            fontFamily: T.font, cursor: "pointer" }}>
          <option value="All">Collection: All</option>
          {collections.map(c => <option key={c} value={c}>{c}</option>)}
        </select>
        <select value={filterType} onChange={e => { setFilterType(e.target.value); setPage(1); }}
          style={{ padding: "5px 10px", borderRadius: T.radius.md, border: `1px solid ${C.border}`,
            background: C.bgCard, fontSize: T.size.md, color: filterType !== "All" ? C.accentBlue : C.textSecondary,
            fontFamily: T.font, cursor: "pointer" }}>
          <option value="All">Type: All</option>
          <option value="PDF">PDF</option><option value="DOCX">DOCX</option><option value="XLSX">XLSX</option>
        </select>
        <select value={sort} onChange={e => setSort(e.target.value)}
          style={{ padding: "5px 10px", borderRadius: T.radius.md, border: `1px solid ${C.border}`,
            background: C.bgCard, fontSize: T.size.md, color: C.textSecondary,
            fontFamily: T.font, cursor: "pointer" }}>
          <option value="updated_desc">Sort: Newest</option>
          <option value="views_desc">Sort: Most Viewed</option>
          <option value="title_asc">Sort: A–Z</option>
        </select>
        {(filterCollection !== "All" || filterType !== "All" || search) && (
          <span onClick={() => { setFilterCollection("All"); setFilterType("All"); handleSearchInput(""); }}
            style={{ color: C.accentBlue, fontSize: T.size.md, cursor: "pointer", padding: "5px 8px" }}>Clear</span>
        )}
      </div>

      {/* Table View */}
      {viewMode === "Table" && (
        <div style={{ flex: 1, overflowY: "auto", padding: "0 24px" }}>
          {loading ? (
            <div style={{ textAlign: "center", padding: 40, color: C.textSecondary }}>Loading...</div>
          ) : docs.length === 0 ? (
            <div style={{ textAlign: "center", padding: 40, color: C.textSecondary }}>No documents found</div>
          ) : (
            <table style={{ width: "100%", borderCollapse: "collapse" }}>
              <thead>
                <tr style={{ borderBottom: `1px solid ${C.border}` }}>
                  {["# ID", "≡ Title", "Collection", "Type", "Status", "Views", "Updated", "Actions"].map((h, i) => (
                    <th key={h} style={{ textAlign: "left", padding: "10px 12px", fontSize: T.size.md,
                      fontWeight: T.weight.semi, color: C.textSecondary, whiteSpace: "nowrap" }}>
                      {h}{i < 6 && <span style={{ color: C.textTertiary, fontSize: 10, marginLeft: 4 }}>↕</span>}
                    </th>
                  ))}
                </tr>
              </thead>
              <tbody>
                {docs.map(a => (
                  <tr key={a.id} style={{ borderBottom: `1px solid ${C.borderLight}`, cursor: "pointer" }}
                    onMouseEnter={e => { for (const td of e.currentTarget.children) td.style.background = C.bgHover; }}
                    onMouseLeave={e => { for (const td of e.currentTarget.children) td.style.background = "transparent"; }}>
                    <td style={{ padding: "12px", fontSize: T.size.md, color: C.accentBlue, fontWeight: T.weight.semi, whiteSpace: "nowrap" }}>#{a.id}</td>
                    <td style={{ padding: "12px", maxWidth: 280 }}>
                      <div style={{ fontWeight: T.weight.medium, fontSize: T.size.md, color: C.textPrimary,
                        whiteSpace: "nowrap", overflow: "hidden", textOverflow: "ellipsis" }}>{a.title}</div>
                      <div style={{ fontSize: T.size.sm, color: C.textSecondary, marginTop: 2 }}>{a.pages} pages · {a.author}</div>
                    </td>
                    <td style={{ padding: "12px", fontSize: T.size.md, color: C.textSecondary }}>
                      <span style={{ display: "flex", alignItems: "center", gap: 6 }}><span style={{ fontSize: 12 }}>📁</span>{a.collection}</span>
                    </td>
                    <td style={{ padding: "12px" }}><TypeBadge type={a.type} /></td>
                    <td style={{ padding: "12px" }}><Badge status={a.status} /></td>
                    <td style={{ padding: "12px", fontSize: T.size.md, color: C.textSecondary }}>{a.views ? a.views.toLocaleString() : "—"}</td>
                    <td style={{ padding: "12px", fontSize: T.size.md, color: C.textSecondary, whiteSpace: "nowrap" }}>({a.updated})</td>
                    <td style={{ padding: "12px", whiteSpace: "nowrap" }} onClick={e => e.stopPropagation()}>
                      <button onClick={() => setEditDoc(a)} style={{ background: "none", border: "none",
                        color: C.accentBlue, cursor: "pointer", fontSize: T.size.sm, marginRight: 8 }}>Edit</button>
                      {a.fileUrl && (
                        <a href={`http://localhost:5000${a.fileUrl}`} download={a.originalName || a.title}
                          style={{ color: C.accentGreen, fontSize: T.size.sm, marginRight: 8, textDecoration: "none" }}>↓</a>
                      )}
                      <button onClick={() => setDeleteConfirm(a)} style={{ background: "none", border: "none",
                        color: C.accentRed, cursor: "pointer", fontSize: T.size.sm }}>Delete</button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          )}
        </div>
      )}

      {/* Grid View */}
      {viewMode === "Grid" && (
        <div style={{ flex: 1, overflowY: "auto", padding: "0 24px", display: "grid",
          gridTemplateColumns: "repeat(auto-fill, minmax(240px, 1fr))", gap: 14, alignContent: "start", paddingBottom: 16 }}>
          {loading ? <div style={{ color: C.textSecondary, padding: 20 }}>Loading...</div> :
            docs.map(a => (
              <div key={a.id} style={{ background: C.bgCard, border: `1px solid ${C.border}`,
                borderRadius: T.radius.lg, padding: 16, cursor: "pointer" }}
                onMouseEnter={e => e.currentTarget.style.background = C.bgHover}
                onMouseLeave={e => e.currentTarget.style.background = C.bgCard}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 10 }}>
                  <TypeBadge type={a.type} />
                  <Badge status={a.status} />
                </div>
                <div style={{ fontWeight: T.weight.semi, fontSize: T.size.base, color: C.textPrimary, marginBottom: 6,
                  overflow: "hidden", display: "-webkit-box", WebkitLineClamp: 2, WebkitBoxOrient: "vertical" }}>{a.title}</div>
                <div style={{ fontSize: T.size.sm, color: C.textSecondary }}>📁 {a.collection}</div>
                <div style={{ fontSize: T.size.sm, color: C.textTertiary, marginTop: 4 }}>{a.author} · {a.pages}p</div>
                <div style={{ display: "flex", justifyContent: "space-between", marginTop: 12, paddingTop: 10, borderTop: `1px solid ${C.borderLight}` }}>
                  <span style={{ fontSize: T.size.xs, color: C.textTertiary }}>{a.views ? `${a.views.toLocaleString()} views` : "No views"}</span>
                  <span onClick={e => { e.stopPropagation(); setEditDoc(a); }} style={{ fontSize: T.size.xs, color: C.accentBlue, cursor: "pointer" }}>Edit</span>
                </div>
              </div>
            ))}
        </div>
      )}

      {/* Pagination */}
      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between",
        padding: "12px 24px", borderTop: `1px solid ${C.border}` }}>
        <div style={{ display: "flex", alignItems: "center", gap: 8, fontSize: T.size.md, color: C.textSecondary }}>
          Show
          <select value={perPage} onChange={e => { setPerPage(Number(e.target.value)); setPage(1); }}
            style={{ background: C.bgInput, border: `1px solid ${C.border}`, borderRadius: T.radius.sm,
              padding: "4px 8px", color: C.textPrimary, fontSize: T.size.md, fontFamily: T.font }}>
            <option value={10}>10</option><option value={25}>25</option><option value={50}>50</option>
          </select>
          per page &nbsp; Showing {total === 0 ? 0 : (page - 1) * perPage + 1}–{Math.min(page * perPage, total)} of {total}
        </div>
        <div style={{ display: "flex", alignItems: "center", gap: 8, fontSize: T.size.md, color: C.textSecondary }}>
          Page
          <button onClick={() => setPage(p => Math.max(1, p - 1))} disabled={page <= 1}
            style={{ background: C.bgCard, border: `1px solid ${C.border}`, borderRadius: T.radius.sm,
              padding: "4px 8px", color: page <= 1 ? C.textTertiary : C.textPrimary, cursor: page <= 1 ? "default" : "pointer" }}>‹</button>
          <span style={{ background: C.bgInput, border: `1px solid ${C.border}`, borderRadius: T.radius.sm,
            padding: "4px 10px", color: C.textPrimary, fontWeight: T.weight.semi }}>{page}</span>
          of {totalPages || 1}
          <button onClick={() => setPage(p => Math.min(totalPages, p + 1))} disabled={page >= totalPages}
            style={{ background: C.bgCard, border: `1px solid ${C.border}`, borderRadius: T.radius.sm,
              padding: "4px 8px", color: page >= totalPages ? C.textTertiary : C.textPrimary, cursor: page >= totalPages ? "default" : "pointer" }}>›</button>
        </div>
      </div>
    </div>
  );
};

// ─── ADD FAQ MODAL ───────────────────────────
const AddFaqModal = ({ onClose, onSuccess, faq }) => {
  const [form, setForm] = useState({ q: faq?.q || "", a: faq?.a || "", cat: faq?.cat || "Billing" });
  const [loading, setLoading] = useState(false);
  const cats = ["Billing", "Technical", "Account", "Product", "General"];

  const submit = async () => {
    if (!form.q || !form.a) return alert("Question and answer are required");
    setLoading(true);
    try {
      const url = faq ? `${API}/faqs/${faq.id}` : `${API}/faqs`;
      const method = faq ? "PUT" : "POST";
      const res = await fetch(url, { method, headers: { "Content-Type": "application/json" }, body: JSON.stringify(form) });
      if (!res.ok) throw new Error("Failed");
      onSuccess(await res.json());
      onClose();
    } catch (e) { alert(e.message); }
    finally { setLoading(false); }
  };

  return (
    <Modal title={faq ? "Edit FAQ" : "Add FAQ"} onClose={onClose}>
      <Field label="Question">
        <Input value={form.q} onChange={e => setForm(p => ({ ...p, q: e.target.value }))} placeholder="Enter the question..." />
      </Field>
      <Field label="Answer">
        <textarea value={form.a} onChange={e => setForm(p => ({ ...p, a: e.target.value }))}
          rows={5} placeholder="Enter the answer..."
          style={{ width: "100%", boxSizing: "border-box", padding: "9px 12px",
            borderRadius: T.radius.md, border: `1px solid ${C.border}`, background: C.bgInput,
            color: C.textPrimary, fontSize: T.size.base, fontFamily: T.font, outline: "none", resize: "vertical" }} />
      </Field>
      <Field label="Category">
        <Select value={form.cat} onChange={e => setForm(p => ({ ...p, cat: e.target.value }))}>
          {cats.map(c => <option key={c} value={c}>{c}</option>)}
        </Select>
      </Field>
      <div style={{ display: "flex", justifyContent: "flex-end", gap: 10, marginTop: 8 }}>
        <Btn variant="secondary" onClick={onClose}>Cancel</Btn>
        <Btn onClick={submit} disabled={loading}>{loading ? "Saving..." : faq ? "Save Changes" : "Add FAQ"}</Btn>
      </div>
    </Modal>
  );
};

// ─── FAQ PANEL ───────────────────────────────
const FaqPanel = ({ onToast }) => {
  const [openId, setOpenId] = useState(null);
  const [cat, setCat] = useState("All");
  const [faqs, setFaqs] = useState([]);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const [searchInput, setSearchInput] = useState("");
  const [showAdd, setShowAdd] = useState(false);
  const [editFaq, setEditFaq] = useState(null);
  const [deleteConfirm, setDeleteConfirm] = useState(null);
  const debounceRef = useRef();
  const FAQ_CATS = ["All", "Billing", "Account", "Product", "Technical"];

  const load = useCallback(async () => {
    setLoading(true);
    try {
      const params = new URLSearchParams({ per_page: 100 });
      if (cat !== "All") params.set("category", cat);
      if (search) params.set("search", search);
      const res = await fetch(`${API}/faqs?${params}`);
      const data = await res.json();
      setFaqs(data.data);
    } catch { onToast("Failed to load FAQs", "error"); }
    finally { setLoading(false); }
  }, [cat, search]);

  useEffect(() => { load(); }, [load]);

  const handleSearch = (v) => {
    setSearchInput(v);
    clearTimeout(debounceRef.current);
    debounceRef.current = setTimeout(() => setSearch(v), 300);
  };

  const deleteFaq = async (id) => {
    try {
      await fetch(`${API}/faqs/${id}`, { method: "DELETE" });
      onToast("FAQ deleted", "success");
      setDeleteConfirm(null);
      load();
    } catch { onToast("Delete failed", "error"); }
  };

  const catColors = { Billing: C.accentYellow, Technical: C.accentRed, Account: C.accentGreen, Product: C.accentBlue };

  return (
    <div>
      {showAdd && <AddFaqModal onClose={() => setShowAdd(false)} onSuccess={() => { load(); onToast("FAQ added!", "success"); }} />}
      {editFaq && <AddFaqModal faq={editFaq} onClose={() => setEditFaq(null)} onSuccess={() => { setEditFaq(null); load(); onToast("FAQ updated!", "success"); }} />}
      {deleteConfirm && (
        <Modal title="Delete FAQ" onClose={() => setDeleteConfirm(null)}>
          <p style={{ color: C.textSecondary, marginBottom: 20 }}>Delete "<strong style={{ color: C.textPrimary }}>{deleteConfirm.q}</strong>"?</p>
          <div style={{ display: "flex", justifyContent: "flex-end", gap: 10 }}>
            <Btn variant="secondary" onClick={() => setDeleteConfirm(null)}>Cancel</Btn>
            <Btn variant="danger" onClick={() => deleteFaq(deleteConfirm.id)}>Delete</Btn>
          </div>
        </Modal>
      )}

      <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between",
        padding: "14px 24px", borderBottom: `1px solid ${C.border}` }}>
        <div style={{ fontSize: T.size.xl, fontWeight: T.weight.bold, color: C.textPrimary }}>FAQs
          <span style={{ marginLeft: 10, fontSize: T.size.base, fontWeight: T.weight.normal, color: C.textSecondary }}>({faqs.length})</span>
        </div>
        <button onClick={() => setShowAdd(true)} style={{ padding: "8px 18px", borderRadius: T.radius.md,
          border: "none", background: C.accentBlue, color: "#fff", fontSize: T.size.base,
          fontWeight: T.weight.semi, cursor: "pointer", fontFamily: T.font }}>+ Add FAQ</button>
      </div>

      <div style={{ padding: "12px 24px" }}>
        <div style={{ display: "flex", alignItems: "center", gap: 8, padding: "8px 14px",
          borderRadius: T.radius.md, border: `1px solid ${C.border}`, background: C.bgInput }}>
          <span style={{ color: C.textTertiary }}>🔍</span>
          <input value={searchInput} onChange={e => handleSearch(e.target.value)}
            placeholder="Search FAQs..." style={{ border: "none", outline: "none",
            background: "transparent", flex: 1, fontSize: T.size.base, fontFamily: T.font, color: C.textPrimary }} />
          {searchInput && <span onClick={() => handleSearch("")} style={{ cursor: "pointer", color: C.textTertiary }}>✕</span>}
        </div>
      </div>

      <div style={{ display: "flex", gap: 0, padding: "0 24px", borderBottom: `1px solid ${C.border}` }}>
        {FAQ_CATS.map(c => (
          <div key={c} onClick={() => setCat(c)} style={{ padding: "10px 16px", fontSize: T.size.md,
            color: cat === c ? C.textWhite : C.textSecondary, cursor: "pointer",
            borderBottom: `2px solid ${cat === c ? C.accentBlue : "transparent"}`,
            marginBottom: -1, fontWeight: cat === c ? T.weight.semi : T.weight.normal }}>{c}</div>
        ))}
      </div>

      <div style={{ padding: "16px 24px", display: "flex", flexDirection: "column", gap: 6 }}>
        {loading ? <div style={{ textAlign: "center", padding: 30, color: C.textSecondary }}>Loading...</div> :
          faqs.length === 0 ? <div style={{ textAlign: "center", padding: 30, color: C.textSecondary }}>No FAQs found</div> :
          faqs.map(f => {
            const cc = catColors[f.cat] || C.textSecondary;
            return (
              <div key={f.id} style={{ border: `1px solid ${C.border}`, borderRadius: T.radius.lg, overflow: "hidden", background: C.bgCard }}>
                <div onClick={() => setOpenId(openId === f.id ? null : f.id)} style={{
                  display: "flex", alignItems: "center", padding: "14px 16px", cursor: "pointer", gap: 12 }}>
                  <span style={{ flex: 1, fontWeight: T.weight.semi, fontSize: T.size.base, color: C.textPrimary }}>{f.q}</span>
                  <span style={{ fontSize: T.size.xs, fontWeight: T.weight.bold, textTransform: "uppercase",
                    letterSpacing: 0.5, padding: "2px 8px", borderRadius: T.radius.sm,
                    background: `${cc}20`, color: cc, flexShrink: 0 }}>{f.cat}</span>
                  <span style={{ fontSize: 12, color: C.textTertiary, transition: "transform 0.2s",
                    transform: openId === f.id ? "rotate(180deg)" : "rotate(0)" }}>▼</span>
                </div>
                {openId === f.id && (
                  <div style={{ borderTop: `1px solid ${C.borderLight}` }}>
                    <div style={{ padding: "12px 16px 8px", fontSize: T.size.md, color: C.textSecondary, lineHeight: 1.7 }}>{f.a}</div>
                    <div style={{ padding: "8px 16px 12px", display: "flex", gap: 10 }}>
                      <button onClick={() => setEditFaq(f)} style={{ background: "none", border: "none",
                        color: C.accentBlue, cursor: "pointer", fontSize: T.size.sm }}>Edit</button>
                      <button onClick={() => setDeleteConfirm(f)} style={{ background: "none", border: "none",
                        color: C.accentRed, cursor: "pointer", fontSize: T.size.sm }}>Delete</button>
                    </div>
                  </div>
                )}
              </div>
            );
          })}
      </div>
    </div>
  );
};

// ─── SEARCH PANEL ────────────────────────────
const SearchPanel = ({ onToast }) => {
  const [query, setQuery] = useState("refund policy");
  const [filter, setFilter] = useState("All");
  const [results, setResults] = useState([]);
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(false);
  const filterOpts = ["All", "Documents", "FAQs"];
  const debounceRef = useRef();
  const inputRef = useRef();

  const doSearch = useCallback(async (q, f) => {
    if (!q.trim()) { setResults([]); setTotal(0); return; }
    setLoading(true);
    try {
      const params = new URLSearchParams({ q, type: f });
      const res = await fetch(`${API}/search?${params}`);
      const data = await res.json();
      setResults(data.data);
      setTotal(data.total);
    } catch { onToast("Search failed", "error"); }
    finally { setLoading(false); }
  }, []);

  useEffect(() => {
    clearTimeout(debounceRef.current);
    debounceRef.current = setTimeout(() => doSearch(query, filter), 300);
  }, [query, filter, doSearch]);

  // Initial load
  useEffect(() => { doSearch("refund policy", "All"); }, []);

  // Keyboard shortcut: / to focus
  useEffect(() => {
    const handler = (e) => { if (e.key === "/" && document.activeElement !== inputRef.current) { e.preventDefault(); inputRef.current?.focus(); } };
    window.addEventListener("keydown", handler);
    return () => window.removeEventListener("keydown", handler);
  }, []);

  const renderSnippet = (snippet) => {
    const parts = snippet.split(/\{|\}/);
    return parts.map((part, i) => i % 2 === 1
      ? <mark key={i} style={{ background: C.accentYellowBg, color: C.accentYellow, borderRadius: 2, padding: "0 3px", fontWeight: T.weight.semi }}>{part}</mark>
      : <span key={i}>{part}</span>
    );
  };

  return (
    <div>
      <div style={{ padding: "14px 24px", borderBottom: `1px solid ${C.border}` }}>
        <div style={{ fontSize: T.size.xl, fontWeight: T.weight.bold, color: C.textPrimary }}>Content Search</div>
      </div>

      <div style={{ padding: "16px 24px 8px" }}>
        <div style={{ display: "flex", alignItems: "center", gap: 8, padding: "10px 16px",
          borderRadius: T.radius.lg, border: `2px solid ${C.border}`, background: C.bgInput }}>
          <span style={{ fontSize: 18, color: C.textTertiary }}>🔍</span>
          <input ref={inputRef} value={query} onChange={e => setQuery(e.target.value)}
            placeholder="Search across all documents, FAQs, and file contents..."
            style={{ border: "none", outline: "none", background: "transparent", flex: 1,
              fontSize: T.size.base, fontFamily: T.font, color: C.textPrimary }} />
          <span style={{ fontSize: T.size.sm, padding: "2px 8px", borderRadius: T.radius.sm,
            border: `1px solid ${C.border}`, color: C.textTertiary, fontFamily: T.fontMono,
            background: C.bgHover }}>/</span>
        </div>
      </div>

      <div style={{ padding: "8px 24px 12px", display: "flex", gap: 6 }}>
        {filterOpts.map(f => (
          <button key={f} onClick={() => setFilter(f)} style={{ padding: "5px 14px", borderRadius: T.radius.pill,
            cursor: "pointer", fontFamily: T.font,
            border: filter === f ? `1px solid ${C.accentBlue}` : `1px solid ${C.border}`,
            background: filter === f ? C.accentBlueBg : C.bgCard,
            color: filter === f ? C.accentBlue : C.textSecondary,
            fontSize: T.size.md, fontWeight: filter === f ? T.weight.semi : T.weight.normal }}>{f}</button>
        ))}
      </div>

      <div style={{ padding: "0 24px 8px", display: "flex", alignItems: "center", justifyContent: "space-between" }}>
        <div style={{ fontSize: T.size.md, color: C.textSecondary }}>
          {loading ? "Searching..." : <>{total} results for <strong style={{ color: C.textPrimary }}>"{query}"</strong></>}
        </div>
        <div style={{ fontSize: T.size.md, color: C.textTertiary }}>Sorted by relevance</div>
      </div>

      <div style={{ padding: "0 24px 20px", display: "flex", flexDirection: "column", gap: 8 }}>
        {results.length === 0 && !loading && query.trim() && (
          <div style={{ textAlign: "center", padding: 30, color: C.textSecondary }}>No results found</div>
        )}
        {results.map(r => (
          <div key={r.id} style={{ border: `1px solid ${C.border}`, borderRadius: T.radius.lg,
            padding: "14px 16px", cursor: "pointer", background: C.bgCard }}
            onMouseEnter={e => e.currentTarget.style.background = C.bgHover}
            onMouseLeave={e => e.currentTarget.style.background = C.bgCard}>
            <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 6 }}>
              <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                <span style={{ fontSize: T.size.xs, fontWeight: T.weight.bold, padding: "2px 8px",
                  borderRadius: T.radius.sm,
                  background: r.type === "FAQ" ? C.accentBlueBg : C.accentGreenBg,
                  color: r.type === "FAQ" ? C.accentBlue : C.accentGreen }}>{r.type}</span>
                <span style={{ fontWeight: T.weight.semi, fontSize: T.size.md, color: C.textPrimary }}>{r.title}</span>
              </div>
              <span style={{ fontSize: T.size.sm, color: C.textTertiary, flexShrink: 0, marginLeft: 12 }}>📁 {r.source}</span>
            </div>
            <div style={{ fontSize: T.size.md, color: C.textSecondary, lineHeight: 1.6 }}>
              {renderSnippet(r.snippet)}
            </div>
            <div style={{ display: "flex", alignItems: "center", gap: 8, marginTop: 10 }}>
              <div style={{ width: 80, height: 4, background: C.bgHover, borderRadius: 2, overflow: "hidden" }}>
                <div style={{ height: "100%", borderRadius: 2, width: `${r.match}%`,
                  background: r.match > 80 ? C.accentGreen : r.match > 50 ? C.accentYellow : C.textTertiary }} />
              </div>
              <span style={{ fontSize: T.size.xs, color: C.textTertiary, fontWeight: T.weight.semi }}>{r.match}% match</span>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};

// ─── MAIN APP ────────────────────────────────
export default function App() {
  const [activePanel, setActivePanel] = useState("documents");
  const [stats, setStats] = useState({ total: 0, faqCount: 0 });
  const [toast, setToast] = useState(null);

  const showToast = useCallback((msg, type = "success") => setToast({ msg, type }), []);

  useEffect(() => {
    const loadStats = async () => {
      try {
        const [docRes, faqRes] = await Promise.all([
          fetch(`${API}/documents/stats`), fetch(`${API}/faqs/count`)
        ]);
        const docStats = await docRes.json();
        const faqData = await faqRes.json();
        setStats({ total: docStats.total, faqCount: faqData.count });
      } catch {}
    };
    loadStats();
    const interval = setInterval(loadStats, 10000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div style={{ fontFamily: T.font, color: C.textPrimary, background: C.bgDeep, minHeight: "100vh", padding: 16 }}>
      {toast && <Toast msg={toast.msg} type={toast.type} onClose={() => setToast(null)} />}
      <div style={{ border: `1px solid ${C.border}`, borderRadius: T.radius.xl, overflow: "hidden",
        display: "flex", minHeight: 620, background: C.bgMain }}>
        <Sidebar activePanel={activePanel} setActivePanel={setActivePanel} stats={stats} />
        <div style={{ flex: 1, display: "flex", flexDirection: "column", minWidth: 0, overflow: "hidden" }}>
          {activePanel === "documents" && <DocumentsPanel onToast={showToast} />}
          {activePanel === "faq" && <FaqPanel onToast={showToast} />}
          {activePanel === "search" && <SearchPanel onToast={showToast} />}
        </div>
      </div>
    </div>
  );
}

