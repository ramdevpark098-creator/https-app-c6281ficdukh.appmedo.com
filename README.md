import { useState, useEffect, useRef } from "react";

// ─── DESIGN TOKENS ────────────────────────────────────────
const C = {
  saffron: "#FF6B00", saffronLight: "#FF8C38", saffronDark: "#D45500",
  teal: "#00897B", tealLight: "#26A69A", tealDark: "#00695C",
  gold: "#FFB300", bg: "#F5F5F0", card: "#FFFFFF",
  text: "#1A1A1A", textSecondary: "#666666", textMuted: "#999999",
  border: "#E8E8E0", success: "#2E7D32", danger: "#C62828", warning: "#E65100",
};

const SCREEN = {
  LOGIN: "login", HOME: "home", JOBS: "jobs", JOB_DETAIL: "job_detail",
  EARNINGS: "earnings", PROFILE: "profile", NOTIFICATIONS: "notifications",
  TRAINING: "training", RATINGS: "ratings",
};

// ─── MOCK API ─────────────────────────────────────────────
// Simulates real network calls with realistic latency and occasional failures
const API = {
  _delay: (ms) => new Promise(res => setTimeout(res, ms)),
  _shouldFail: () => Math.random() < 0.06, // 6% failure rate for realism

  async login(phone, otp) {
    await this._delay(1400 + Math.random() * 600);
    if (phone === "9999999999" && otp !== "1234") throw new Error("Invalid OTP. Please try again.");
    if (phone.length !== 10) throw new Error("Phone number must be 10 digits.");
    if (otp !== "1234") throw new Error("Invalid OTP. Please try again.");
    return { success: true, partner: { name: "Ramesh Shinde", phone } };
  },

  async sendOtp(phone) {
    await this._delay(800 + Math.random() * 400);
    if (this._shouldFail()) throw new Error("Network error. Please try again.");
    if (phone.length !== 10 || !/^\d+$/.test(phone)) throw new Error("Enter a valid 10-digit mobile number.");
    return { success: true, message: "OTP sent to +91 " + phone };
  },

  async acceptJob(jobId) {
    await this._delay(900 + Math.random() * 500);
    if (this._shouldFail()) throw new Error("Server busy. Please try again.");
    return { success: true, jobId, message: "Job accepted! Customer has been notified." };
  },

  async declineJob(jobId, reason) {
    await this._delay(700 + Math.random() * 300);
    return { success: true, jobId, message: "Job declined." };
  },

  async markComplete(jobId) {
    await this._delay(1100 + Math.random() * 400);
    if (this._shouldFail()) throw new Error("Could not update status. Retrying…");
    return { success: true, jobId, message: "Job marked complete! Payment processing." };
  },

  async updateAvailability(isOnline) {
    await this._delay(500 + Math.random() * 300);
    return { success: true, isOnline, message: isOnline ? "You are now online." : "You are now offline." };
  },

  async updateProfile(data) {
    await this._delay(1200 + Math.random() * 500);
    if (!data.name?.trim()) throw new Error("Name cannot be empty.");
    if (data.phone && data.phone.length !== 10) throw new Error("Enter a valid phone number.");
    return { success: true, message: "Profile updated successfully!" };
  },
};

// ─── SEED DATA ────────────────────────────────────────────
const INIT_JOBS = [
  { id: 1, service: "Deep Home Cleaning", customer: "Priya Sharma", location: "Baner, Pune", time: "Today 10:00 AM", amount: 1200, status: "new", distance: "2.3 km", phone: "98765 43210", address: "Flat 204, Orchid Heights, Baner", notes: "3BHK flat, please bring all equipment", category: "cleaning" },
  { id: 2, service: "AC Repair & Service", customer: "Rajesh Patil", location: "Wakad, Pune", time: "Today 2:00 PM", amount: 800, status: "new", distance: "4.1 km", phone: "99887 76655", address: "Plot 12, Lotus Colony, Wakad", notes: "Split AC not cooling", category: "repair" },
  { id: 3, service: "Bathroom Plumbing Fix", customer: "Sunita Desai", location: "Kothrud, Pune", time: "Tomorrow 9:00 AM", amount: 600, status: "accepted", distance: "5.8 km", phone: "97654 32100", address: "Lane 5, Prabhat Road, Kothrud", notes: "Tap leaking and drain blocked", category: "plumbing" },
  { id: 4, service: "Salon at Home – Bridal", customer: "Anita Joshi", location: "Aundh, Pune", time: "Today 4:00 PM", amount: 3500, status: "accepted", distance: "3.2 km", phone: "90123 45678", address: "Sector 7, Aundh", notes: "Full bridal makeup + saree draping", category: "salon" },
  { id: 5, service: "Pest Control – Cockroach", customer: "Mahesh Kulkarni", location: "Hadapsar, Pune", time: "Yesterday 11:00 AM", amount: 950, status: "completed", distance: "7.4 km", phone: "87654 32198", address: "Magarpatta City, Hadapsar", notes: "", category: "pest" },
  { id: 6, service: "Sofa Deep Cleaning", customer: "Deepa Naik", location: "Viman Nagar", time: "Jun 3, 3:00 PM", amount: 700, status: "completed", distance: "6.0 km", phone: "91234 56789", address: "Blue Ridge, Viman Nagar", notes: "3 seater + 1 single", category: "cleaning" },
];

const EARNINGS_DATA = {
  today: 2000, week: 11450, month: 38200, pending: 1800,
  chart: [4200, 5800, 3900, 7100, 6400, 8200, 11450],
  days: ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Today"],
  transactions: [
    { id: "T001", job: "Bridal Salon", customer: "Anita Joshi", amount: 3500, date: "Today", status: "pending" },
    { id: "T002", job: "Deep Cleaning", customer: "Suresh Mehta", amount: 1200, date: "Yesterday", status: "paid" },
    { id: "T003", job: "AC Repair", customer: "Kavita More", amount: 800, date: "Jun 4", status: "paid" },
    { id: "T004", job: "Pest Control", customer: "Mahesh Kulkarni", amount: 950, date: "Jun 3", status: "paid" },
    { id: "T005", job: "Sofa Cleaning", customer: "Deepa Naik", amount: 700, date: "Jun 3", status: "paid" },
  ],
};

const NOTIFICATIONS_INIT = [
  { id: 1, type: "job", title: "New Job Request!", message: "Deep Home Cleaning in Baner – ₹1,200", time: "2 min ago", read: false },
  { id: 2, type: "payment", title: "Payment Received", message: "₹1,200 for Sofa Cleaning credited to your wallet", time: "1 hr ago", read: false },
  { id: 3, type: "rating", title: "New Review", message: "Priya Sharma gave you 5 ⭐ — 'Excellent work!'", time: "3 hrs ago", read: true },
  { id: 4, type: "promo", title: "Bonus Offer!", message: "Complete 5 jobs this week and earn ₹500 bonus", time: "Today, 8:00 AM", read: true },
  { id: 5, type: "job", title: "Job Reminder", message: "You have AC Repair at Wakad at 2:00 PM today", time: "Yesterday", read: true },
];

const RATINGS_DATA = [
  { customer: "Priya Sharma", service: "Deep Cleaning", rating: 5, comment: "Excellent work! Very thorough and professional.", date: "Today" },
  { customer: "Suresh Mehta", service: "Bathroom Plumbing", rating: 5, comment: "Fixed everything quickly. Highly recommended!", date: "Yesterday" },
  { customer: "Kavita More", service: "AC Service", rating: 4, comment: "Good service, arrived on time.", date: "Jun 4" },
  { customer: "Deepa Naik", service: "Sofa Cleaning", rating: 5, comment: "Sofa looks brand new! Great job.", date: "Jun 3" },
  { customer: "Rahul Joshi", service: "Pest Control", rating: 4, comment: "Effective treatment, no issues since.", date: "Jun 2" },
];

const catColor = { cleaning: "#00897B", repair: "#E65100", plumbing: "#1565C0", salon: "#AD1457", pest: "#6A1B9A" };
const catLabel = { cleaning: "Cleaning", repair: "Repair", plumbing: "Plumbing", salon: "Salon", pest: "Pest Control" };

// ─── ICON ────────────────────────────────────────────────
const Icon = ({ name, size = 20, color = C.text, style = {} }) => {
  const icons = {
    home: <path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/>,
    briefcase: <><rect x="2" y="7" width="20" height="14" rx="2"/><path d="M16 7V5a2 2 0 00-2-2h-4a2 2 0 00-2 2v2"/></>,
    rupee: <path d="M6 3h12M6 8h12m-7 5l7 8M6 8a4 4 0 004 4h2.5"/>,
    user: <><path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/><circle cx="12" cy="7" r="4"/></>,
    bell: <><path d="M18 8A6 6 0 006 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 01-3.46 0"/></>,
    star: <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/>,
    check: <polyline points="20 6 9 17 4 12"/>,
    x: <><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></>,
    clock: <><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></>,
    location: <><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0118 0z"/><circle cx="12" cy="10" r="3"/></>,
    phone: <path d="M22 16.92v3a2 2 0 01-2.18 2 19.79 19.79 0 01-8.63-3.07A19.5 19.5 0 013.42 13.5a19.79 19.79 0 01-3.07-8.67A2 2 0 012.37 2.71h3a2 2 0 012 1.72c.127.96.361 1.903.7 2.81a2 2 0 01-.45 2.11L6.91 10.1a16 16 0 006 6l1.27-1.27a2 2 0 012.11-.45c.907.339 1.85.573 2.81.7a2 2 0 011.72 2.03z"/>,
    chat: <path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z"/>,
    arrow_right: <><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></>,
    arrow_left: <><line x1="19" y1="12" x2="5" y2="12"/><polyline points="12 19 5 12 12 5"/></>,
    trending_up: <><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></>,
    shield: <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>,
    award: <><circle cx="12" cy="8" r="6"/><path d="M15.477 12.89L17 22l-5-3-5 3 1.523-9.11"/></>,
    tool: <path d="M14.7 6.3a1 1 0 000 1.4l1.6 1.6a1 1 0 001.4 0l3.77-3.77a6 6 0 01-7.94 7.94l-6.91 6.91a2.12 2.12 0 01-3-3l6.91-6.91a6 6 0 017.94-7.94l-3.76 3.76z"/>,
    calendar: <><rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></>,
    settings: <><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 012.83 2.83l-.06.06A1.65 1.65 0 0019.4 9a1.65 1.65 0 001.51 1H21a2 2 0 010 4h-.09a1.65 1.65 0 00-1.51 1z"/></>,
    logout: <><path d="M9 21H5a2 2 0 01-2-2V5a2 2 0 012-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></>,
    info: <><circle cx="12" cy="12" r="10"/><line x1="12" y1="16" x2="12" y2="12"/><line x1="12" y1="8" x2="12.01" y2="8"/></>,
    zap: <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>,
    wallet: <><rect x="1" y="4" width="22" height="16" rx="2" ry="2"/><line x1="1" y1="10" x2="23" y2="10"/></>,
    refresh: <><polyline points="23 4 23 10 17 10"/><path d="M20.49 15a9 9 0 11-2.12-9.36L23 10"/></>,
    edit: <><path d="M11 4H4a2 2 0 00-2 2v14a2 2 0 002 2h14a2 2 0 002-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 013 3L12 15l-4 1 1-4 9.5-9.5z"/></>,
  };
  return (
    <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color}
      strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" style={style}>
      {icons[name]}
    </svg>
  );
};

// ─── REUSABLE ATOMS ──────────────────────────────────────
const StatusBadge = ({ status }) => {
  const config = {
    new: { bg: "#FFF3E0", color: C.warning, label: "New" },
    accepted: { bg: "#E8F5E9", color: C.success, label: "Accepted" },
    completed: { bg: "#E3F2FD", color: "#1565C0", label: "Done" },
    pending: { bg: "#FFF8E1", color: "#F57F17", label: "Pending" },
    paid: { bg: "#E8F5E9", color: C.success, label: "Paid" },
    declined: { bg: "#FFEBEE", color: C.danger, label: "Declined" },
  };
  const c = config[status] || config.new;
  return (
    <span style={{ background: c.bg, color: c.color, padding: "2px 10px", borderRadius: 20, fontSize: 11, fontWeight: 700, letterSpacing: 0.5 }}>
      {c.label}
    </span>
  );
};

const StarRating = ({ rating, size = 14 }) => (
  <div style={{ display: "flex", gap: 2 }}>
    {[1,2,3,4,5].map(s => (
      <svg key={s} width={size} height={size} viewBox="0 0 24 24"
        fill={s <= rating ? C.gold : "#E0E0E0"} stroke={s <= rating ? C.gold : "#E0E0E0"} strokeWidth="1">
        <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/>
      </svg>
    ))}
  </div>
);

// Toast notification system
const Toast = ({ toasts, removeToast }) => (
  <div style={{ position: "absolute", top: 52, left: 12, right: 12, zIndex: 999, display: "flex", flexDirection: "column", gap: 6, pointerEvents: "none" }}>
    {toasts.map(t => (
      <div key={t.id} style={{
        background: t.type === "error" ? "#FFEBEE" : t.type === "warning" ? "#FFF8E1" : "#E8F5E9",
        border: `1px solid ${t.type === "error" ? C.danger + "40" : t.type === "warning" ? C.gold + "50" : C.success + "40"}`,
        borderRadius: 12, padding: "10px 14px", display: "flex", gap: 8, alignItems: "flex-start",
        boxShadow: "0 4px 16px rgba(0,0,0,0.12)", pointerEvents: "all",
        animation: "slideDown 0.25s ease",
      }}>
        <span style={{ fontSize: 16, flexShrink: 0 }}>
          {t.type === "error" ? "❌" : t.type === "warning" ? "⚠️" : "✅"}
        </span>
        <span style={{ fontSize: 13, fontWeight: 600, color: C.text, lineHeight: 1.4, flex: 1 }}>{t.message}</span>
        <button onClick={() => removeToast(t.id)} style={{ background: "none", border: "none", cursor: "pointer", padding: 0, flexShrink: 0 }}>
          <Icon name="x" size={14} color={C.textMuted} />
        </button>
      </div>
    ))}
  </div>
);

// Spinner
const Spinner = ({ size = 20, color = "#fff" }) => (
  <div style={{
    width: size, height: size, border: `2px solid ${color}40`,
    borderTop: `2px solid ${color}`, borderRadius: "50%",
    animation: "spin 0.7s linear infinite", display: "inline-block",
  }} />
);

// API Status indicator (shows mock API call activity)
const ApiIndicator = ({ calls }) => {
  if (calls.length === 0) return null;
  return (
    <div style={{
      position: "absolute", bottom: 90, left: 12, right: 12, zIndex: 998,
      background: "rgba(0,0,0,0.82)", borderRadius: 10, padding: "7px 12px",
      backdropFilter: "blur(8px)", display: "flex", alignItems: "center", gap: 8,
    }}>
      <Spinner size={13} color={C.saffron} />
      <span style={{ fontSize: 11, color: "rgba(255,255,255,0.85)", fontFamily: "monospace" }}>
        {calls[calls.length - 1]}
      </span>
    </div>
  );
};

// ─── LOGIN SCREEN ─────────────────────────────────────────
const LoginScreen = ({ onLogin, addToast, addApiCall, removeApiCall }) => {
  const [step, setStep] = useState("phone"); // phone | otp
  const [phone, setPhone] = useState("");
  const [otp, setOtp] = useState("");
  const [loading, setLoading] = useState(false);
  const [errors, setErrors] = useState({});
  const [otpSent, setOtpSent] = useState(false);
  const [resendTimer, setResendTimer] = useState(0);
  const otpRefs = [useRef(), useRef(), useRef(), useRef()];

  // Resend countdown
  useEffect(() => {
    if (resendTimer > 0) {
      const t = setTimeout(() => setResendTimer(r => r - 1), 1000);
      return () => clearTimeout(t);
    }
  }, [resendTimer]);

  const validatePhone = () => {
    if (!phone) return "Mobile number is required.";
    if (!/^\d{10}$/.test(phone)) return "Enter a valid 10-digit mobile number.";
    return null;
  };

  const handleSendOtp = async () => {
    const err = validatePhone();
    if (err) { setErrors({ phone: err }); return; }
    setErrors({});
    setLoading(true);
    const callId = "POST /auth/send-otp";
    addApiCall(callId);
    try {
      await API.sendOtp(phone);
      setOtpSent(true);
      setStep("otp");
      setResendTimer(30);
      addToast({ type: "success", message: `OTP sent to +91 ${phone}. Use 1234 for demo.` });
    } catch (e) {
      addToast({ type: "error", message: e.message });
      setErrors({ phone: e.message });
    } finally {
      setLoading(false);
      removeApiCall(callId);
    }
  };

  const handleOtpChange = (val, idx) => {
    const digits = val.replace(/\D/g, "").slice(0, 1);
    const arr = otp.split("").concat(Array(4).fill("")).slice(0, 4);
    arr[idx] = digits;
    const newOtp = arr.join("").slice(0, 4);
    setOtp(newOtp);
    if (digits && idx < 3) otpRefs[idx + 1].current?.focus();
    if (!digits && idx > 0) otpRefs[idx - 1].current?.focus();
  };

  const handleLogin = async () => {
    if (otp.length !== 4) { setErrors({ otp: "Enter the 4-digit OTP." }); return; }
    setErrors({});
    setLoading(true);
    const callId = "POST /auth/verify-otp";
    addApiCall(callId);
    try {
      const res = await API.login(phone, otp);
      addToast({ type: "success", message: "Login successful! Welcome back 🙏" });
      setTimeout(() => onLogin(res.partner), 400);
    } catch (e) {
      addToast({ type: "error", message: e.message });
      setErrors({ otp: e.message });
      setOtp("");
      otpRefs[0].current?.focus();
    } finally {
      setLoading(false);
      removeApiCall(callId);
    }
  };

  const handleResend = async () => {
    if (resendTimer > 0) return;
    setLoading(true);
    const callId = "POST /auth/resend-otp";
    addApiCall(callId);
    try {
      await API.sendOtp(phone);
      setResendTimer(30);
      setOtp("");
      addToast({ type: "success", message: "OTP resent successfully." });
    } catch (e) {
      addToast({ type: "error", message: e.message });
    } finally {
      setLoading(false);
      removeApiCall(callId);
    }
  };

  return (
    <div style={{ minHeight: "100%", background: `linear-gradient(160deg, ${C.saffronDark} 0%, ${C.saffron} 45%, #FF8C38 100%)`, display: "flex", flexDirection: "column" }}>
      {/* Top art */}
      <div style={{ padding: "60px 28px 32px", position: "relative", overflow: "hidden" }}>
        <div style={{ position: "absolute", top: -60, right: -60, width: 200, height: 200, borderRadius: "50%", background: "rgba(255,255,255,0.08)" }} />
        <div style={{ position: "absolute", top: 20, right: 30, width: 100, height: 100, borderRadius: "50%", background: "rgba(255,255,255,0.06)" }} />
        <div style={{ fontSize: 48, marginBottom: 12 }}>🔧</div>
        <div style={{ color: "#fff", fontSize: 26, fontWeight: 900, letterSpacing: -0.5, lineHeight: 1.2 }}>
          MahaMaintenance<br />
          <span style={{ color: "rgba(255,255,255,0.85)" }}>Pro Partner</span>
        </div>
        <div style={{ color: "rgba(255,255,255,0.75)", fontSize: 13, marginTop: 8 }}>Pune's trusted service professional network</div>
      </div>

      {/* Card */}
      <div style={{ flex: 1, background: C.bg, borderRadius: "28px 28px 0 0", padding: "28px 24px 24px", display: "flex", flexDirection: "column", gap: 20 }}>
        {step === "phone" ? (
          <>
            <div>
              <div style={{ fontSize: 20, fontWeight: 800, color: C.text }}>Welcome back 🙏</div>
              <div style={{ fontSize: 13, color: C.textSecondary, marginTop: 4 }}>Enter your registered mobile number to continue</div>
            </div>
            <div>
              <div style={{ fontSize: 12, fontWeight: 700, color: C.textSecondary, marginBottom: 8, letterSpacing: 0.3 }}>MOBILE NUMBER</div>
              <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
                <div style={{ padding: "13px 14px", background: C.card, border: `1.5px solid ${C.border}`, borderRadius: 14, fontSize: 15, fontWeight: 700, color: C.text }}>+91</div>
                <input
                  type="tel" maxLength={10} value={phone}
                  onChange={e => { setPhone(e.target.value.replace(/\D/g, "").slice(0, 10)); setErrors({}); }}
                  onKeyDown={e => e.key === "Enter" && handleSendOtp()}
                  placeholder="98765 43210"
                  style={{
                    flex: 1, padding: "13px 16px", background: C.card,
                    border: `1.5px solid ${errors.phone ? C.danger : C.border}`, borderRadius: 14,
                    fontSize: 18, fontWeight: 700, color: C.text, outline: "none", letterSpacing: 1,
                    fontFamily: "monospace",
                  }}
                />
              </div>
              {errors.phone && <div style={{ color: C.danger, fontSize: 12, marginTop: 6, fontWeight: 600 }}>⚠ {errors.phone}</div>}
            </div>

            <div style={{ background: "#FFF8E1", borderRadius: 12, padding: "10px 14px", border: `1px solid ${C.gold}40`, fontSize: 12, color: C.warning }}>
              <strong>Demo:</strong> Use any 10-digit number + OTP <strong>1234</strong>
            </div>

            <button onClick={handleSendOtp} disabled={loading}
              style={{ padding: "15px", background: loading ? C.saffron + "80" : C.saffron, color: "#fff", border: "none", borderRadius: 14, fontSize: 16, fontWeight: 800, cursor: loading ? "not-allowed" : "pointer", display: "flex", alignItems: "center", justifyContent: "center", gap: 8, boxShadow: `0 4px 16px ${C.saffron}50` }}>
              {loading ? <><Spinner /><span>Sending OTP…</span></> : "Send OTP →"}
            </button>
          </>
        ) : (
          <>
            <div>
              <button onClick={() => { setStep("phone"); setOtp(""); setErrors({}); }}
                style={{ background: "none", border: "none", cursor: "pointer", display: "flex", alignItems: "center", gap: 6, color: C.saffron, fontWeight: 700, fontSize: 13, marginBottom: 16, padding: 0 }}>
                <Icon name="arrow_left" size={16} color={C.saffron} /> Change number
              </button>
              <div style={{ fontSize: 20, fontWeight: 800, color: C.text }}>Enter OTP</div>
              <div style={{ fontSize: 13, color: C.textSecondary, marginTop: 4 }}>
                Sent to <strong>+91 {phone}</strong>
              </div>
            </div>

            {/* OTP boxes */}
            <div style={{ display: "flex", gap: 10, justifyContent: "center" }}>
              {[0,1,2,3].map(i => (
                <input
                  key={i} ref={otpRefs[i]}
                  type="tel" maxLength={1}
                  value={otp[i] || ""}
                  onChange={e => handleOtpChange(e.target.value, i)}
                  onKeyDown={e => {
                    if (e.key === "Backspace" && !otp[i] && i > 0) otpRefs[i - 1].current?.focus();
                    if (e.key === "Enter" && otp.length === 4) handleLogin();
                  }}
                  style={{
                    width: 58, height: 62, textAlign: "center", fontSize: 26, fontWeight: 800,
                    background: C.card, border: `2px solid ${errors.otp ? C.danger : otp[i] ? C.saffron : C.border}`,
                    borderRadius: 14, color: C.text, outline: "none", fontFamily: "monospace",
                    transition: "border-color 0.15s",
                  }}
                />
              ))}
            </div>
            {errors.otp && <div style={{ color: C.danger, fontSize: 12, textAlign: "center", fontWeight: 600 }}>⚠ {errors.otp}</div>}

            <div style={{ textAlign: "center", fontSize: 13 }}>
              {resendTimer > 0
                ? <span style={{ color: C.textMuted }}>Resend OTP in <strong style={{ color: C.text }}>{resendTimer}s</strong></span>
                : <button onClick={handleResend} disabled={loading}
                    style={{ background: "none", border: "none", cursor: "pointer", color: C.saffron, fontWeight: 700, fontSize: 13 }}>
                    Resend OTP
                  </button>
              }
            </div>

            <button onClick={handleLogin} disabled={loading || otp.length !== 4}
              style={{ padding: "15px", background: otp.length === 4 ? C.saffron : C.border, color: otp.length === 4 ? "#fff" : C.textMuted, border: "none", borderRadius: 14, fontSize: 16, fontWeight: 800, cursor: otp.length === 4 && !loading ? "pointer" : "not-allowed", display: "flex", alignItems: "center", justifyContent: "center", gap: 8, transition: "all 0.2s", boxShadow: otp.length === 4 ? `0 4px 16px ${C.saffron}50` : "none" }}>
              {loading ? <><Spinner /><span>Verifying…</span></> : "Verify & Login →"}
            </button>
          </>
        )}
      </div>
    </div>
  );
};

// ─── HOME SCREEN ──────────────────────────────────────────
const HomeScreen = ({ setScreen, setSelectedJob, jobs, partner, isOnline, toggleOnline, addToast, addApiCall, removeApiCall }) => {
  const [toggling, setToggling] = useState(false);
  const newJobs = jobs.filter(j => j.status === "new");
  const todayJobs = jobs.filter(j => j.status === "accepted" && j.time.includes("Today"));

  const handleToggle = async () => {
    setToggling(true);
    const callId = `PATCH /partner/availability`;
    addApiCall(callId);
    try {
      const res = await API.updateAvailability(!isOnline);
      toggleOnline();
      addToast({ type: "success", message: res.message });
    } catch (e) {
      addToast({ type: "error", message: e.message });
    } finally {
      setToggling(false);
      removeApiCall(callId);
    }
  };

  return (
    <div style={{ padding: "0 0 16px" }}>
      <div style={{ background: `linear-gradient(135deg, ${C.saffron} 0%, ${C.saffronDark} 100%)`, padding: "52px 20px 24px", position: "relative", overflow: "hidden" }}>
        <div style={{ position: "absolute", top: -40, right: -40, width: 160, height: 160, borderRadius: "50%", background: "rgba(255,255,255,0.08)" }} />
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", position: "relative" }}>
          <div>
            <div style={{ color: "rgba(255,255,255,0.85)", fontSize: 13, marginBottom: 4 }}>Namaste 🙏</div>
            <div style={{ color: "#fff", fontSize: 22, fontWeight: 800, letterSpacing: -0.5 }}>{partner.name}</div>
            <div style={{ display: "flex", alignItems: "center", gap: 6, marginTop: 6 }}>
              <Icon name="star" size={14} color={C.gold} style={{ fill: C.gold }} />
              <span style={{ color: "#fff", fontSize: 14, fontWeight: 700 }}>{partner.rating}</span>
              <span style={{ color: "rgba(255,255,255,0.7)", fontSize: 12 }}>({partner.reviews} reviews)</span>
            </div>
          </div>
          {/* Online toggle */}
          <button onClick={handleToggle} disabled={toggling}
            style={{ display: "flex", flexDirection: "column", alignItems: "center", background: "rgba(255,255,255,0.15)", border: "none", borderRadius: 16, padding: "10px 14px", cursor: "pointer", backdropFilter: "blur(8px)" }}>
            {toggling
              ? <Spinner size={18} color="#fff" />
              : <span style={{ color: "#fff", fontSize: 11, fontWeight: 600 }}>{isOnline ? "Online" : "Offline"}</span>
            }
            <div style={{ width: 36, height: 20, background: isOnline ? C.success : C.textMuted, borderRadius: 10, marginTop: 4, display: "flex", alignItems: "center", padding: "0 3px", transition: "background 0.3s", justifyContent: isOnline ? "flex-end" : "flex-start" }}>
              <div style={{ width: 14, height: 14, borderRadius: "50%", background: "#fff" }} />
            </div>
          </button>
        </div>

        <div style={{ display: "flex", gap: 10, marginTop: 20 }}>
          {[
            { label: "Today's Earnings", value: `₹${EARNINGS_DATA.today.toLocaleString()}` },
            { label: "Jobs Done", value: partner.completedJobs },
            { label: "Pending", value: newJobs.length },
          ].map((s, i) => (
            <div key={i} style={{ flex: 1, background: "rgba(255,255,255,0.15)", backdropFilter: "blur(8px)", borderRadius: 14, padding: "10px 8px", textAlign: "center" }}>
              <div style={{ color: "rgba(255,255,255,0.8)", fontSize: 10, marginBottom: 2 }}>{s.label}</div>
              <div style={{ color: "#fff", fontSize: 18, fontWeight: 800 }}>{s.value}</div>
            </div>
          ))}
        </div>
      </div>

      <div style={{ padding: "0 16px" }}>
        {!isOnline && (
          <div style={{ marginTop: 16, background: "#FFF3E0", border: `1px solid ${C.warning}30`, borderRadius: 14, padding: "12px 14px", display: "flex", alignItems: "center", gap: 10 }}>
            <span style={{ fontSize: 18 }}>😴</span>
            <div>
              <div style={{ fontSize: 13, fontWeight: 700, color: C.warning }}>You're offline</div>
              <div style={{ fontSize: 12, color: C.textSecondary }}>Go online to start receiving job requests.</div>
            </div>
          </div>
        )}

        {newJobs.length > 0 && (
          <div style={{ marginTop: 20 }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
              <span style={{ fontSize: 16, fontWeight: 800, color: C.text }}>🔔 New Requests</span>
              <span style={{ fontSize: 12, color: C.saffron, fontWeight: 700 }}>{newJobs.length} new</span>
            </div>
            <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
              {newJobs.map(job => (
                <div key={job.id} onClick={() => { setSelectedJob(job); setScreen(SCREEN.JOB_DETAIL); }}
                  style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 12px rgba(0,0,0,0.08)", border: `2px solid ${C.saffron}20`, cursor: "pointer", position: "relative", overflow: "hidden" }}>
                  <div style={{ position: "absolute", top: 0, left: 0, width: 4, height: "100%", background: C.saffron }} />
                  <div style={{ paddingLeft: 8 }}>
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                      <div>
                        <div style={{ fontSize: 15, fontWeight: 800, color: C.text }}>{job.service}</div>
                        <div style={{ fontSize: 12, color: C.textSecondary, marginTop: 2 }}>{job.customer}</div>
                      </div>
                      <div style={{ textAlign: "right" }}>
                        <div style={{ fontSize: 18, fontWeight: 900, color: C.saffron }}>₹{job.amount}</div>
                        <StatusBadge status={job.status} />
                      </div>
                    </div>
                    <div style={{ display: "flex", gap: 16, marginTop: 10 }}>
                      <div style={{ display: "flex", alignItems: "center", gap: 4 }}>
                        <Icon name="clock" size={13} color={C.textMuted} />
                        <span style={{ fontSize: 12, color: C.textSecondary }}>{job.time}</span>
                      </div>
                      <div style={{ display: "flex", alignItems: "center", gap: 4 }}>
                        <Icon name="location" size={13} color={C.textMuted} />
                        <span style={{ fontSize: 12, color: C.textSecondary }}>{job.distance}</span>
                      </div>
                    </div>
                    <div style={{ fontSize: 11, color: C.textMuted, marginTop: 6, fontStyle: "italic" }}>Tap to view details & respond</div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {todayJobs.length > 0 && (
          <div style={{ marginTop: 24 }}>
            <div style={{ fontSize: 16, fontWeight: 800, color: C.text, marginBottom: 12 }}>📅 Today's Schedule</div>
            <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
              {todayJobs.map(job => (
                <div key={job.id} onClick={() => { setSelectedJob(job); setScreen(SCREEN.JOB_DETAIL); }}
                  style={{ background: C.card, borderRadius: 16, padding: 14, boxShadow: "0 2px 8px rgba(0,0,0,0.06)", cursor: "pointer", display: "flex", alignItems: "center", gap: 12 }}>
                  <div style={{ width: 44, height: 44, borderRadius: 12, background: `${catColor[job.category]}20`, display: "flex", alignItems: "center", justifyContent: "center" }}>
                    <Icon name="tool" size={20} color={catColor[job.category]} />
                  </div>
                  <div style={{ flex: 1 }}>
                    <div style={{ fontSize: 14, fontWeight: 700, color: C.text }}>{job.service}</div>
                    <div style={{ fontSize: 12, color: C.textSecondary }}>{job.time} · {job.location}</div>
                  </div>
                  <div style={{ fontSize: 15, fontWeight: 800, color: C.teal }}>₹{job.amount}</div>
                </div>
              ))}
            </div>
          </div>
        )}

        <div style={{ marginTop: 24, marginBottom: 8 }}>
          <div style={{ fontSize: 16, fontWeight: 800, color: C.text, marginBottom: 12 }}>Quick Actions</div>
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10 }}>
            {[
              { icon: "wallet", label: "My Earnings", screen: SCREEN.EARNINGS, color: C.teal },
              { icon: "star", label: "My Ratings", screen: SCREEN.RATINGS, color: C.gold },
              { icon: "award", label: "Training", screen: SCREEN.TRAINING, color: "#7B1FA2" },
              { icon: "settings", label: "Profile", screen: SCREEN.PROFILE, color: C.saffron },
            ].map((a, i) => (
              <button key={i} onClick={() => setScreen(a.screen)}
                style={{ background: C.card, border: "none", borderRadius: 16, padding: 16, display: "flex", alignItems: "center", gap: 10, cursor: "pointer", boxShadow: "0 2px 8px rgba(0,0,0,0.06)", textAlign: "left" }}>
                <div style={{ width: 40, height: 40, borderRadius: 12, background: `${a.color}18`, display: "flex", alignItems: "center", justifyContent: "center" }}>
                  <Icon name={a.icon} size={20} color={a.color} />
                </div>
                <span style={{ fontSize: 13, fontWeight: 700, color: C.text }}>{a.label}</span>
              </button>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
};

// ─── JOBS SCREEN ──────────────────────────────────────────
const JobsScreen = ({ setScreen, setSelectedJob, jobs }) => {
  const [filter, setFilter] = useState("all");
  const filters = [
    { key: "all", label: "All" }, { key: "new", label: "New" },
    { key: "accepted", label: "Upcoming" }, { key: "completed", label: "Done" }, { key: "declined", label: "Declined" },
  ];
  const filtered = filter === "all" ? jobs : jobs.filter(j => j.status === filter);

  return (
    <div style={{ padding: "0 0 16px" }}>
      <div style={{ padding: "52px 20px 16px", background: `linear-gradient(135deg, ${C.teal} 0%, ${C.tealDark} 100%)` }}>
        <div style={{ color: "#fff", fontSize: 22, fontWeight: 800 }}>My Jobs</div>
        <div style={{ color: "rgba(255,255,255,0.75)", fontSize: 13, marginTop: 2 }}>{jobs.length} total assignments</div>
      </div>

      <div style={{ display: "flex", gap: 8, padding: "14px 16px", overflowX: "auto", scrollbarWidth: "none" }}>
        {filters.map(f => {
          const count = f.key === "all" ? jobs.length : jobs.filter(j => j.status === f.key).length;
          if (count === 0 && f.key !== "all" && f.key !== filter) return null;
          return (
            <button key={f.key} onClick={() => setFilter(f.key)}
              style={{ padding: "8px 18px", borderRadius: 20, border: "none", cursor: "pointer", whiteSpace: "nowrap",
                background: filter === f.key ? C.teal : C.card, color: filter === f.key ? "#fff" : C.textSecondary,
                fontWeight: filter === f.key ? 700 : 500, fontSize: 13,
                boxShadow: filter === f.key ? `0 2px 8px ${C.teal}40` : "0 1px 4px rgba(0,0,0,0.06)" }}>
              {f.label} {filter === f.key ? `(${filtered.length})` : count > 0 ? `(${count})` : ""}
            </button>
          );
        })}
      </div>

      <div style={{ padding: "0 16px", display: "flex", flexDirection: "column", gap: 12 }}>
        {filtered.length === 0
          ? <div style={{ textAlign: "center", padding: "40px 20px", color: C.textMuted }}>
              <div style={{ fontSize: 40, marginBottom: 8 }}>📭</div>
              <div style={{ fontSize: 14, fontWeight: 600 }}>No {filter === "all" ? "" : filter} jobs</div>
            </div>
          : filtered.map(job => (
            <div key={job.id} onClick={() => { setSelectedJob(job); setScreen(SCREEN.JOB_DETAIL); }}
              style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.07)", cursor: "pointer", opacity: job.status === "declined" ? 0.7 : 1 }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 10 }}>
                <div style={{ display: "flex", gap: 10, alignItems: "center" }}>
                  <div style={{ width: 40, height: 40, borderRadius: 12, background: `${catColor[job.category]}18`, display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }}>
                    <Icon name="tool" size={18} color={catColor[job.category]} />
                  </div>
                  <div>
                    <div style={{ fontSize: 15, fontWeight: 800, color: C.text }}>{job.service}</div>
                    <div style={{ fontSize: 12, color: C.textSecondary }}>{catLabel[job.category]}</div>
                  </div>
                </div>
                <StatusBadge status={job.status} />
              </div>
              <div style={{ height: 1, background: C.border, margin: "10px 0" }} />
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                <div>
                  <div style={{ fontSize: 13, fontWeight: 700, color: C.text }}>{job.customer}</div>
                  <div style={{ display: "flex", gap: 12, marginTop: 4 }}>
                    <span style={{ fontSize: 11, color: C.textMuted, display: "flex", alignItems: "center", gap: 3 }}>
                      <Icon name="clock" size={11} color={C.textMuted} />{job.time}
                    </span>
                    <span style={{ fontSize: 11, color: C.textMuted, display: "flex", alignItems: "center", gap: 3 }}>
                      <Icon name="location" size={11} color={C.textMuted} />{job.location}
                    </span>
                  </div>
                </div>
                <div style={{ fontSize: 18, fontWeight: 900, color: job.status === "completed" ? C.success : job.status === "declined" ? C.textMuted : C.saffron }}>
                  {job.status !== "declined" ? `₹${job.amount}` : "—"}
                </div>
              </div>
            </div>
          ))
        }
      </div>
    </div>
  );
};

// ─── JOB DETAIL SCREEN ────────────────────────────────────
const JobDetailScreen = ({ job, setScreen, onJobUpdate, addToast, addApiCall, removeApiCall }) => {
  const [status, setStatus] = useState(job.status);
  const [loading, setLoading] = useState(null); // "accept" | "decline" | "complete"
  const [showDeclineModal, setShowDeclineModal] = useState(false);
  const [declineReason, setDeclineReason] = useState("");
  const [declineError, setDeclineError] = useState("");

  if (!job) return null;

  const handleAccept = async () => {
    setLoading("accept");
    const callId = `POST /jobs/${job.id}/accept`;
    addApiCall(callId);
    try {
      await API.acceptJob(job.id);
      setStatus("accepted");
      onJobUpdate(job.id, "accepted");
      addToast({ type: "success", message: "Job accepted! Customer notified." });
    } catch (e) {
      addToast({ type: "error", message: e.message });
    } finally {
      setLoading(null);
      removeApiCall(callId);
    }
  };

  const handleDecline = async () => {
    if (!declineReason.trim()) { setDeclineError("Please select or enter a reason."); return; }
    setDeclineError("");
    setLoading("decline");
    const callId = `POST /jobs/${job.id}/decline`;
    addApiCall(callId);
    try {
      await API.declineJob(job.id, declineReason);
      setStatus("declined");
      onJobUpdate(job.id, "declined");
      setShowDeclineModal(false);
      addToast({ type: "warning", message: "Job declined." });
    } catch (e) {
      addToast({ type: "error", message: e.message });
    } finally {
      setLoading(null);
      removeApiCall(callId);
    }
  };

  const handleComplete = async () => {
    setLoading("complete");
    const callId = `PATCH /jobs/${job.id}/complete`;
    addApiCall(callId);
    try {
      await API.markComplete(job.id);
      setStatus("completed");
      onJobUpdate(job.id, "completed");
      addToast({ type: "success", message: "Job completed! Payment of ₹" + job.amount + " processing." });
    } catch (e) {
      addToast({ type: "error", message: e.message });
    } finally {
      setLoading(null);
      removeApiCall(callId);
    }
  };

  const declineReasons = ["Schedule conflict", "Too far from location", "Outside my service area", "Service not available today", "Other"];

  return (
    <div style={{ background: C.bg, minHeight: "100%" }}>
      {/* Decline modal */}
      {showDeclineModal && (
        <div style={{ position: "absolute", inset: 0, background: "rgba(0,0,0,0.5)", zIndex: 998, display: "flex", alignItems: "flex-end" }}>
          <div style={{ background: C.card, borderRadius: "20px 20px 0 0", padding: 24, width: "100%", boxSizing: "border-box" }}>
            <div style={{ fontSize: 17, fontWeight: 800, color: C.text, marginBottom: 4 }}>Decline Job</div>
            <div style={{ fontSize: 13, color: C.textSecondary, marginBottom: 16 }}>Please tell us why you're declining this job.</div>
            <div style={{ display: "flex", flexDirection: "column", gap: 8, marginBottom: 12 }}>
              {declineReasons.map(r => (
                <button key={r} onClick={() => setDeclineReason(r)}
                  style={{ padding: "11px 14px", background: declineReason === r ? `${C.danger}12` : C.bg, border: `1.5px solid ${declineReason === r ? C.danger : C.border}`, borderRadius: 12, textAlign: "left", fontSize: 13, fontWeight: declineReason === r ? 700 : 500, color: declineReason === r ? C.danger : C.text, cursor: "pointer" }}>
                  {r}
                </button>
              ))}
            </div>
            {declineError && <div style={{ color: C.danger, fontSize: 12, fontWeight: 600, marginBottom: 8 }}>⚠ {declineError}</div>}
            <div style={{ display: "flex", gap: 10 }}>
              <button onClick={() => { setShowDeclineModal(false); setDeclineReason(""); setDeclineError(""); }}
                style={{ flex: 1, padding: "13px", background: C.bg, border: "none", borderRadius: 13, fontSize: 14, fontWeight: 700, cursor: "pointer", color: C.textSecondary }}>
                Cancel
              </button>
              <button onClick={handleDecline} disabled={loading === "decline"}
                style={{ flex: 1, padding: "13px", background: C.danger, border: "none", borderRadius: 13, fontSize: 14, fontWeight: 700, cursor: "pointer", color: "#fff", display: "flex", alignItems: "center", justifyContent: "center", gap: 6 }}>
                {loading === "decline" ? <><Spinner /><span>Declining…</span></> : "Confirm Decline"}
              </button>
            </div>
          </div>
        </div>
      )}

      <div style={{ background: `linear-gradient(135deg, ${catColor[job.category]} 0%, ${catColor[job.category]}cc 100%)`, padding: "52px 20px 24px" }}>
        <button onClick={() => setScreen(SCREEN.JOBS)} style={{ background: "rgba(255,255,255,0.2)", border: "none", borderRadius: 12, padding: "8px 12px", display: "flex", alignItems: "center", gap: 6, color: "#fff", cursor: "pointer", marginBottom: 16 }}>
          <Icon name="arrow_left" size={16} color="#fff" />
          <span style={{ fontSize: 13, fontWeight: 600 }}>Back</span>
        </button>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
          <div>
            <div style={{ fontSize: 20, fontWeight: 800, color: "#fff" }}>{job.service}</div>
            <div style={{ fontSize: 13, color: "rgba(255,255,255,0.8)", marginTop: 4 }}>{catLabel[job.category]}</div>
            <div style={{ fontSize: 28, fontWeight: 900, color: "#fff", marginTop: 8 }}>₹{job.amount}</div>
          </div>
          <StatusBadge status={status} />
        </div>
      </div>

      <div style={{ padding: 16, display: "flex", flexDirection: "column", gap: 14 }}>
        {/* Customer */}
        <div style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
          <div style={{ fontSize: 12, fontWeight: 700, color: C.textMuted, marginBottom: 10, textTransform: "uppercase", letterSpacing: 0.5 }}>Customer</div>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
            <div>
              <div style={{ fontSize: 17, fontWeight: 800, color: C.text }}>{job.customer}</div>
              <div style={{ fontSize: 13, color: C.textSecondary, marginTop: 2 }}>{job.phone}</div>
            </div>
            <div style={{ display: "flex", gap: 8 }}>
              <button style={{ width: 42, height: 42, borderRadius: 12, background: `${C.teal}18`, border: "none", display: "flex", alignItems: "center", justifyContent: "center", cursor: "pointer" }}>
                <Icon name="phone" size={18} color={C.teal} />
              </button>
              <button style={{ width: 42, height: 42, borderRadius: 12, background: `${C.saffron}18`, border: "none", display: "flex", alignItems: "center", justifyContent: "center", cursor: "pointer" }}>
                <Icon name="chat" size={18} color={C.saffron} />
              </button>
            </div>
          </div>
        </div>

        {/* Location */}
        <div style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
          <div style={{ fontSize: 12, fontWeight: 700, color: C.textMuted, marginBottom: 10, textTransform: "uppercase", letterSpacing: 0.5 }}>Location</div>
          <div style={{ display: "flex", gap: 10 }}>
            <Icon name="location" size={18} color={C.saffron} />
            <div>
              <div style={{ fontSize: 14, fontWeight: 700, color: C.text }}>{job.address}</div>
              <div style={{ fontSize: 12, color: C.textSecondary, marginTop: 2 }}>{job.distance} from you</div>
            </div>
          </div>
          <button style={{ width: "100%", marginTop: 12, padding: "10px", background: `${C.teal}15`, border: `1px solid ${C.teal}30`, borderRadius: 12, color: C.teal, fontWeight: 700, fontSize: 13, cursor: "pointer" }}>
            📍 Open in Maps
          </button>
        </div>

        {/* Schedule */}
        <div style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
          <div style={{ fontSize: 12, fontWeight: 700, color: C.textMuted, marginBottom: 10, textTransform: "uppercase", letterSpacing: 0.5 }}>Schedule</div>
          <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
            <Icon name="calendar" size={18} color={C.saffron} />
            <div style={{ fontSize: 15, fontWeight: 700, color: C.text }}>{job.time}</div>
          </div>
        </div>

        {/* Notes */}
        {job.notes && (
          <div style={{ background: "#FFF8E1", borderRadius: 16, padding: 14, border: `1px solid ${C.gold}30` }}>
            <div style={{ display: "flex", gap: 8, alignItems: "flex-start" }}>
              <Icon name="info" size={16} color={C.gold} />
              <div>
                <div style={{ fontSize: 12, fontWeight: 700, color: C.warning, marginBottom: 4 }}>Customer Notes</div>
                <div style={{ fontSize: 13, color: C.text }}>{job.notes}</div>
              </div>
            </div>
          </div>
        )}

        {/* Actions */}
        {status === "new" && (
          <div style={{ display: "flex", gap: 10 }}>
            <button onClick={handleAccept} disabled={!!loading}
              style={{ flex: 2, padding: "14px", background: loading === "accept" ? C.saffron + "80" : C.saffron, color: "#fff", border: "none", borderRadius: 14, fontSize: 15, fontWeight: 800, cursor: loading ? "not-allowed" : "pointer", boxShadow: `0 4px 14px ${C.saffron}50`, display: "flex", alignItems: "center", justifyContent: "center", gap: 8 }}>
              {loading === "accept" ? <><Spinner /><span>Accepting…</span></> : "✓ Accept Job"}
            </button>
            <button onClick={() => setShowDeclineModal(true)} disabled={!!loading}
              style={{ flex: 1, padding: "14px", background: C.card, color: C.danger, border: `2px solid ${C.danger}30`, borderRadius: 14, fontSize: 15, fontWeight: 700, cursor: loading ? "not-allowed" : "pointer" }}>
              Decline
            </button>
          </div>
        )}
        {status === "accepted" && (
          <button onClick={handleComplete} disabled={!!loading}
            style={{ width: "100%", padding: "14px", background: loading === "complete" ? C.success + "80" : C.success, color: "#fff", border: "none", borderRadius: 14, fontSize: 15, fontWeight: 800, cursor: loading ? "not-allowed" : "pointer", boxShadow: `0 4px 14px ${C.success}50`, display: "flex", alignItems: "center", justifyContent: "center", gap: 8 }}>
            {loading === "complete" ? <><Spinner /><span>Updating…</span></> : "✓ Mark as Completed"}
          </button>
        )}
        {status === "completed" && (
          <div style={{ background: "#E8F5E9", borderRadius: 14, padding: 16, textAlign: "center", border: `1px solid ${C.success}30` }}>
            <div style={{ fontSize: 24, marginBottom: 6 }}>🎉</div>
            <div style={{ fontSize: 16, fontWeight: 800, color: C.success }}>Job Completed!</div>
            <div style={{ fontSize: 12, color: C.textSecondary, marginTop: 4 }}>₹{job.amount} will be credited within 24 hours</div>
          </div>
        )}
        {status === "declined" && (
          <div style={{ background: "#FFEBEE", borderRadius: 14, padding: 14, textAlign: "center", border: `1px solid ${C.danger}20` }}>
            <div style={{ fontSize: 14, fontWeight: 700, color: C.danger }}>Job Declined</div>
            <div style={{ fontSize: 12, color: C.textSecondary, marginTop: 4 }}>This job has been returned to the pool.</div>
          </div>
        )}
      </div>
    </div>
  );
};

// ─── EARNINGS SCREEN ──────────────────────────────────────
const EarningsScreen = () => {
  const maxVal = Math.max(...EARNINGS_DATA.chart);
  const [period, setPeriod] = useState("week");
  const displayAmount = period === "today" ? EARNINGS_DATA.today : period === "week" ? EARNINGS_DATA.week : EARNINGS_DATA.month;

  return (
    <div style={{ padding: "0 0 16px" }}>
      <div style={{ background: `linear-gradient(135deg, #1B5E20 0%, #2E7D32 100%)`, padding: "52px 20px 28px", position: "relative", overflow: "hidden" }}>
        <div style={{ position: "absolute", top: -30, right: -30, width: 120, height: 120, borderRadius: "50%", background: "rgba(255,255,255,0.07)" }} />
        <div style={{ color: "rgba(255,255,255,0.8)", fontSize: 13, marginBottom: 4 }}>Total Earnings</div>
        <div style={{ color: "#fff", fontSize: 36, fontWeight: 900 }}>₹{displayAmount.toLocaleString()}</div>
        <div style={{ display: "flex", gap: 8, marginTop: 16 }}>
          {["today", "week", "month"].map(p => (
            <button key={p} onClick={() => setPeriod(p)}
              style={{ padding: "6px 14px", borderRadius: 20, border: "none", cursor: "pointer",
                background: period === p ? "rgba(255,255,255,0.25)" : "rgba(255,255,255,0.1)",
                color: "#fff", fontWeight: period === p ? 700 : 400, fontSize: 12 }}>
              {p.charAt(0).toUpperCase() + p.slice(1)}
            </button>
          ))}
        </div>
      </div>

      <div style={{ padding: "16px" }}>
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, marginBottom: 20 }}>
          {[
            { label: "Pending Payout", value: `₹${EARNINGS_DATA.pending}`, color: C.warning, icon: "clock" },
            { label: "This Month", value: `₹${EARNINGS_DATA.month.toLocaleString()}`, color: C.success, icon: "trending_up" },
          ].map((s, i) => (
            <div key={i} style={{ background: C.card, borderRadius: 16, padding: 14, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
              <div style={{ width: 32, height: 32, borderRadius: 10, background: `${s.color}18`, display: "flex", alignItems: "center", justifyContent: "center", marginBottom: 8 }}>
                <Icon name={s.icon} size={16} color={s.color} />
              </div>
              <div style={{ fontSize: 18, fontWeight: 900, color: s.color }}>{s.value}</div>
              <div style={{ fontSize: 11, color: C.textMuted, marginTop: 2 }}>{s.label}</div>
            </div>
          ))}
        </div>

        <div style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)", marginBottom: 16 }}>
          <div style={{ fontSize: 14, fontWeight: 800, color: C.text, marginBottom: 16 }}>Weekly Overview</div>
          <div style={{ display: "flex", alignItems: "flex-end", gap: 8, height: 90 }}>
            {EARNINGS_DATA.chart.map((val, i) => {
              const h = (val / maxVal) * 80;
              const isLast = i === EARNINGS_DATA.chart.length - 1;
              return (
                <div key={i} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", gap: 4 }}>
                  <div style={{ width: "100%", height: h, background: isLast ? C.saffron : `${C.teal}60`, borderRadius: "6px 6px 0 0" }} />
                  <span style={{ fontSize: 9, color: isLast ? C.saffron : C.textMuted, fontWeight: isLast ? 700 : 400 }}>{EARNINGS_DATA.days[i]}</span>
                </div>
              );
            })}
          </div>
        </div>

        <div style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
          <div style={{ fontSize: 14, fontWeight: 800, color: C.text, marginBottom: 14 }}>Recent Transactions</div>
          <div style={{ display: "flex", flexDirection: "column", gap: 0 }}>
            {EARNINGS_DATA.transactions.map((t, i) => (
              <div key={t.id}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "10px 0" }}>
                  <div>
                    <div style={{ fontSize: 14, fontWeight: 700, color: C.text }}>{t.job}</div>
                    <div style={{ fontSize: 12, color: C.textSecondary }}>{t.customer} · {t.date}</div>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <div style={{ fontSize: 15, fontWeight: 800, color: C.success }}>+₹{t.amount}</div>
                    <StatusBadge status={t.status} />
                  </div>
                </div>
                {i < EARNINGS_DATA.transactions.length - 1 && <div style={{ height: 1, background: C.border }} />}
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
};

// ─── NOTIFICATIONS SCREEN ─────────────────────────────────
const NotificationsScreen = ({ notifications, markAllRead }) => (
  <div>
    <div style={{ padding: "52px 20px 20px", background: `linear-gradient(135deg, #4A148C 0%, #6A1B9A 100%)`, display: "flex", justifyContent: "space-between", alignItems: "flex-end" }}>
      <div>
        <div style={{ color: "#fff", fontSize: 22, fontWeight: 800 }}>Notifications</div>
        <div style={{ color: "rgba(255,255,255,0.75)", fontSize: 13, marginTop: 2 }}>{notifications.filter(n => !n.read).length} unread</div>
      </div>
      <button onClick={markAllRead} style={{ background: "rgba(255,255,255,0.15)", border: "none", borderRadius: 10, padding: "7px 12px", color: "#fff", fontSize: 12, fontWeight: 600, cursor: "pointer" }}>
        Mark all read
      </button>
    </div>
    <div style={{ padding: "12px 16px", display: "flex", flexDirection: "column", gap: 10 }}>
      {notifications.map(n => {
        const typeConfig = {
          job: { color: C.saffron, icon: "briefcase" },
          payment: { color: C.success, icon: "wallet" },
          rating: { color: C.gold, icon: "star" },
          promo: { color: "#7B1FA2", icon: "zap" },
        };
        const cfg = typeConfig[n.type];
        return (
          <div key={n.id} style={{ background: n.read ? C.card : `${cfg.color}08`, borderRadius: 16, padding: 14, boxShadow: "0 2px 8px rgba(0,0,0,0.06)", border: n.read ? "none" : `1px solid ${cfg.color}25`, display: "flex", gap: 12 }}>
            <div style={{ width: 40, height: 40, borderRadius: 12, background: `${cfg.color}18`, display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }}>
              <Icon name={cfg.icon} size={18} color={cfg.color} />
            </div>
            <div style={{ flex: 1 }}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                <div style={{ fontSize: 14, fontWeight: n.read ? 600 : 800, color: C.text }}>{n.title}</div>
                {!n.read && <div style={{ width: 8, height: 8, borderRadius: "50%", background: cfg.color, flexShrink: 0, marginTop: 4 }} />}
              </div>
              <div style={{ fontSize: 12, color: C.textSecondary, marginTop: 3 }}>{n.message}</div>
              <div style={{ fontSize: 11, color: C.textMuted, marginTop: 6 }}>{n.time}</div>
            </div>
          </div>
        );
      })}
    </div>
  </div>
);

// ─── RATINGS SCREEN ───────────────────────────────────────
const RatingsScreen = () => {
  const avg = (RATINGS_DATA.reduce((s, r) => s + r.rating, 0) / RATINGS_DATA.length).toFixed(1);
  return (
    <div>
      <div style={{ background: `linear-gradient(135deg, #F57F17 0%, ${C.gold} 100%)`, padding: "52px 20px 24px" }}>
        <div style={{ color: "#fff", fontSize: 22, fontWeight: 800 }}>My Ratings</div>
        <div style={{ display: "flex", alignItems: "center", gap: 12, marginTop: 12 }}>
          <div style={{ fontSize: 52, fontWeight: 900, color: "#fff", lineHeight: 1 }}>{avg}</div>
          <div>
            <StarRating rating={Math.round(avg)} size={22} />
            <div style={{ color: "rgba(255,255,255,0.85)", fontSize: 12, marginTop: 4 }}>{RATINGS_DATA.length} reviews</div>
          </div>
        </div>
      </div>
      <div style={{ padding: "16px", display: "flex", flexDirection: "column", gap: 12 }}>
        {RATINGS_DATA.map((r, i) => (
          <div key={i} style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 8 }}>
              <div>
                <div style={{ fontSize: 15, fontWeight: 800, color: C.text }}>{r.customer}</div>
                <div style={{ fontSize: 12, color: C.textSecondary }}>{r.service}</div>
              </div>
              <div style={{ textAlign: "right" }}>
                <StarRating rating={r.rating} />
                <div style={{ fontSize: 11, color: C.textMuted, marginTop: 3 }}>{r.date}</div>
              </div>
            </div>
            {r.comment && (
              <div style={{ background: C.bg, borderRadius: 10, padding: "8px 12px", fontSize: 13, color: C.textSecondary, fontStyle: "italic" }}>
                "{r.comment}"
              </div>
            )}
          </div>
        ))}
      </div>
    </div>
  );
};

// ─── TRAINING SCREEN ──────────────────────────────────────
const TrainingScreen = () => {
  const courses = [
    { title: "Advanced Cleaning Techniques", modules: 8, duration: "2.5 hrs", progress: 75, color: C.teal },
    { title: "Customer Service Excellence", modules: 6, duration: "1.5 hrs", progress: 100, color: C.success },
    { title: "Safety & PPE Guidelines", modules: 5, duration: "1 hr", progress: 40, color: C.warning },
    { title: "Festival Service Special", modules: 10, duration: "3 hrs", progress: 0, color: "#7B1FA2" },
    { title: "Premium Salon Techniques", modules: 12, duration: "4 hrs", progress: 20, color: C.saffron },
  ];
  return (
    <div>
      <div style={{ background: `linear-gradient(135deg, #4527A0 0%, #7B1FA2 100%)`, padding: "52px 20px 24px" }}>
        <div style={{ color: "#fff", fontSize: 22, fontWeight: 800 }}>Training Hub</div>
        <div style={{ color: "rgba(255,255,255,0.8)", fontSize: 13, marginTop: 2 }}>Level up your skills & earn more</div>
        <div style={{ display: "flex", gap: 10, marginTop: 16 }}>
          {[{ label: "Completed", value: "1" }, { label: "In Progress", value: "3" }, { label: "Certificates", value: "1" }].map((s, i) => (
            <div key={i} style={{ flex: 1, background: "rgba(255,255,255,0.15)", borderRadius: 12, padding: "10px 8px", textAlign: "center" }}>
              <div style={{ color: "#fff", fontSize: 20, fontWeight: 800 }}>{s.value}</div>
              <div style={{ color: "rgba(255,255,255,0.75)", fontSize: 11 }}>{s.label}</div>
            </div>
          ))}
        </div>
      </div>
      <div style={{ padding: "16px", display: "flex", flexDirection: "column", gap: 12 }}>
        {courses.map((c, i) => (
          <div key={i} style={{ background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
            <div style={{ display: "flex", gap: 12, alignItems: "flex-start" }}>
              <div style={{ width: 44, height: 44, borderRadius: 12, background: `${c.color}18`, display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }}>
                <Icon name="award" size={22} color={c.color} />
              </div>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 14, fontWeight: 800, color: C.text }}>{c.title}</div>
                <div style={{ fontSize: 12, color: C.textSecondary, marginTop: 2 }}>{c.modules} modules · {c.duration}</div>
                <div style={{ marginTop: 10 }}>
                  <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 4 }}>
                    <span style={{ fontSize: 11, color: C.textMuted }}>Progress</span>
                    <span style={{ fontSize: 11, fontWeight: 700, color: c.color }}>{c.progress}%</span>
                  </div>
                  <div style={{ height: 6, background: C.border, borderRadius: 3 }}>
                    <div style={{ height: "100%", width: `${c.progress}%`, background: c.color, borderRadius: 3 }} />
                  </div>
                </div>
                <button style={{ marginTop: 10, padding: "7px 14px", background: c.progress === 100 ? `${C.success}18` : `${c.color}18`, border: "none", borderRadius: 10, color: c.progress === 100 ? C.success : c.color, fontWeight: 700, fontSize: 12, cursor: "pointer" }}>
                  {c.progress === 0 ? "Start Course" : c.progress === 100 ? "✓ Completed" : "Continue"}
                </button>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};

// ─── PROFILE SCREEN ───────────────────────────────────────
const ProfileScreen = ({ partner, onLogout, addToast, addApiCall, removeApiCall }) => {
  const [editing, setEditing] = useState(false);
  const [form, setForm] = useState({ name: partner.name, phone: partner.phone || "98765 43210", city: partner.city });
  const [errors, setErrors] = useState({});
  const [saving, setSaving] = useState(false);

  const validate = () => {
    const e = {};
    if (!form.name.trim()) e.name = "Name is required.";
    else if (form.name.trim().length < 2) e.name = "Name must be at least 2 characters.";
    const rawPhone = form.phone.replace(/\s/g, "");
    if (!rawPhone) e.phone = "Phone number is required.";
    else if (!/^\d{10}$/.test(rawPhone)) e.phone = "Enter a valid 10-digit number.";
    if (!form.city.trim()) e.city = "City is required.";
    return e;
  };

  const handleSave = async () => {
    const e = validate();
    if (Object.keys(e).length) { setErrors(e); return; }
    setErrors({});
    setSaving(true);
    const callId = "PATCH /partner/profile";
    addApiCall(callId);
    try {
      await API.updateProfile(form);
      setEditing(false);
      addToast({ type: "success", message: "Profile updated successfully!" });
    } catch (err) {
      addToast({ type: "error", message: err.message });
      setErrors({ api: err.message });
    } finally {
      setSaving(false);
      removeApiCall(callId);
    }
  };

  const Field = ({ label, field, keyboardType = "text" }) => (
    <div style={{ marginBottom: 14 }}>
      <div style={{ fontSize: 11, fontWeight: 700, color: C.textMuted, marginBottom: 6, letterSpacing: 0.4 }}>{label.toUpperCase()}</div>
      <input
        value={form[field]}
        onChange={e => { setForm(f => ({ ...f, [field]: e.target.value })); setErrors(er => ({ ...er, [field]: null })); }}
        type={keyboardType}
        style={{
          width: "100%", padding: "11px 14px", background: C.bg,
          border: `1.5px solid ${errors[field] ? C.danger : C.border}`, borderRadius: 12,
          fontSize: 14, fontWeight: 600, color: C.text, outline: "none", boxSizing: "border-box",
        }}
      />
      {errors[field] && <div style={{ color: C.danger, fontSize: 12, marginTop: 4, fontWeight: 600 }}>⚠ {errors[field]}</div>}
    </div>
  );

  const menuItems = [
    { icon: "briefcase", label: "My Services", sub: "8 services active", color: C.saffron },
    { icon: "wallet", label: "Bank Account", sub: "SBI ****4521", color: C.success },
    { icon: "shield", label: "Background Check", sub: "Verified ✓", color: C.teal },
    { icon: "bell", label: "Notifications", sub: "Manage alerts", color: "#7B1FA2" },
    { icon: "info", label: "Help & Support", sub: "FAQs, contact us", color: "#1565C0" },
    { icon: "settings", label: "Settings", sub: "Language, privacy", color: C.textSecondary },
    { icon: "logout", label: "Logout", sub: "", color: C.danger, action: onLogout },
  ];

  return (
    <div style={{ paddingBottom: 20 }}>
      <div style={{ background: `linear-gradient(135deg, ${C.saffronDark} 0%, ${C.saffron} 100%)`, padding: "52px 20px 32px", textAlign: "center", position: "relative" }}>
        <button onClick={() => setEditing(e => !e)}
          style={{ position: "absolute", top: 52, right: 16, background: "rgba(255,255,255,0.2)", border: "none", borderRadius: 10, padding: "8px 12px", color: "#fff", cursor: "pointer", display: "flex", alignItems: "center", gap: 5, fontSize: 12, fontWeight: 700 }}>
          <Icon name="edit" size={14} color="#fff" />
          {editing ? "Cancel" : "Edit"}
        </button>
        <div style={{ width: 80, height: 80, borderRadius: "50%", background: "#fff", margin: "0 auto 12px", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 32, boxShadow: "0 4px 16px rgba(0,0,0,0.2)" }}>👷</div>
        <div style={{ color: "#fff", fontSize: 20, fontWeight: 800 }}>{partner.name}</div>
        <div style={{ color: "rgba(255,255,255,0.8)", fontSize: 13, marginTop: 2 }}>{partner.service} · {partner.city}</div>
        <div style={{ display: "flex", justifyContent: "center", alignItems: "center", gap: 6, marginTop: 8 }}>
          <StarRating rating={5} size={14} />
          <span style={{ color: "#fff", fontSize: 13, fontWeight: 700 }}>{partner.rating} ({partner.reviews} reviews)</span>
        </div>
        <div style={{ display: "flex", gap: 10, marginTop: 16, justifyContent: "center" }}>
          {[
            { value: partner.completedJobs, label: "Jobs Done" },
            { value: "3", label: "Badges" },
            { value: "96%", label: "Completion" },
          ].map((s, i) => (
            <div key={i} style={{ background: "rgba(255,255,255,0.2)", borderRadius: 12, padding: "10px 20px", textAlign: "center" }}>
              <div style={{ color: "#fff", fontSize: 20, fontWeight: 900 }}>{s.value}</div>
              <div style={{ color: "rgba(255,255,255,0.8)", fontSize: 11 }}>{s.label}</div>
            </div>
          ))}
        </div>
      </div>

      {/* Edit form */}
      {editing && (
        <div style={{ margin: "14px 16px 0", background: C.card, borderRadius: 16, padding: 16, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
          <div style={{ fontSize: 14, fontWeight: 800, color: C.text, marginBottom: 14 }}>Edit Profile</div>
          <Field label="Full Name" field="name" />
          <Field label="Mobile Number" field="phone" keyboardType="tel" />
          <Field label="City" field="city" />
          {errors.api && <div style={{ color: C.danger, fontSize: 12, fontWeight: 600, marginBottom: 10 }}>⚠ {errors.api}</div>}
          <button onClick={handleSave} disabled={saving}
            style={{ width: "100%", padding: "13px", background: saving ? C.saffron + "80" : C.saffron, color: "#fff", border: "none", borderRadius: 13, fontSize: 14, fontWeight: 800, cursor: saving ? "not-allowed" : "pointer", display: "flex", alignItems: "center", justifyContent: "center", gap: 8 }}>
            {saving ? <><Spinner /><span>Saving…</span></> : "Save Changes"}
          </button>
        </div>
      )}

      {/* Badges */}
      <div style={{ margin: "14px 16px 0", background: C.card, borderRadius: 16, padding: 14, boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
        <div style={{ fontSize: 12, fontWeight: 700, color: C.textMuted, marginBottom: 10, letterSpacing: 0.4 }}>MY BADGES</div>
        <div style={{ display: "flex", gap: 10 }}>
          {[
            { emoji: "⭐", label: "Top Rated", color: C.gold },
            { emoji: "🚀", label: "Fast Pro", color: C.teal },
            { emoji: "💯", label: "100 Jobs", color: C.saffron },
          ].map((b, i) => (
            <div key={i} style={{ textAlign: "center" }}>
              <div style={{ width: 48, height: 48, borderRadius: 14, background: `${b.color}18`, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 22, margin: "0 auto 4px" }}>{b.emoji}</div>
              <div style={{ fontSize: 10, color: b.color, fontWeight: 700 }}>{b.label}</div>
            </div>
          ))}
        </div>
      </div>

      {/* Menu */}
      <div style={{ margin: "14px 16px 0", background: C.card, borderRadius: 16, overflow: "hidden", boxShadow: "0 2px 10px rgba(0,0,0,0.06)" }}>
        {menuItems.map((item, i) => (
          <div key={i}>
            <div onClick={item.action} style={{ display: "flex", alignItems: "center", gap: 12, padding: "14px 16px", cursor: item.action ? "pointer" : "default" }}>
              <div style={{ width: 36, height: 36, borderRadius: 10, background: `${item.color}18`, display: "flex", alignItems: "center", justifyContent: "center" }}>
                <Icon name={item.icon} size={18} color={item.color} />
              </div>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 14, fontWeight: 700, color: item.label === "Logout" ? C.danger : C.text }}>{item.label}</div>
                {item.sub && <div style={{ fontSize: 11, color: C.textMuted }}>{item.sub}</div>}
              </div>
              {item.label !== "Logout" && <Icon name="arrow_right" size={16} color={C.textMuted} />}
            </div>
            {i < menuItems.length - 1 && <div style={{ height: 1, background: C.border, marginLeft: 64 }} />}
          </div>
        ))}
      </div>
    </div>
  );
};

// ─── MAIN APP ─────────────────────────────────────────────
export default function MahaMaintenanceProPartner() {
  const [screen, setScreen] = useState(SCREEN.LOGIN);
  const [selectedJob, setSelectedJob] = useState(null);
  const [jobs, setJobs] = useState(INIT_JOBS);
  const [activeTab, setActiveTab] = useState("home");
  const [loggedIn, setLoggedIn] = useState(false);
  const [isOnline, setIsOnline] = useState(true);
  const [notifications, setNotifications] = useState(NOTIFICATIONS_INIT);
  const [toasts, setToasts] = useState([]);
  const [apiCalls, setApiCalls] = useState([]);
  const toastId = useRef(0);

  const partner = {
    name: "Ramesh Shinde",
    service: "Multi-Service Expert",
    city: "Pune",
    rating: "4.9",
    reviews: 287,
    completedJobs: jobs.filter(j => j.status === "completed").length + 306,
    phone: "98765 43210",
  };

  const addToast = (t) => {
    const id = ++toastId.current;
    setToasts(prev => [...prev, { ...t, id }]);
    setTimeout(() => removeToast(id), 4000);
  };
  const removeToast = (id) => setToasts(prev => prev.filter(t => t.id !== id));
  const addApiCall = (call) => setApiCalls(prev => [...prev, call]);
  const removeApiCall = (call) => setApiCalls(prev => prev.filter(c => c !== call));

  const handleLogin = (partnerData) => {
    setLoggedIn(true);
    setScreen(SCREEN.HOME);
    setActiveTab("home");
  };
  const handleLogout = () => {
    setLoggedIn(false);
    setScreen(SCREEN.LOGIN);
    setActiveTab("home");
    setToasts([]);
  };

  const handleJobUpdate = (jobId, newStatus) => {
    setJobs(prev => prev.map(j => j.id === jobId ? { ...j, status: newStatus } : j));
    if (selectedJob?.id === jobId) setSelectedJob(prev => ({ ...prev, status: newStatus }));
  };

  const markAllRead = () => setNotifications(prev => prev.map(n => ({ ...n, read: true })));
  const unreadNotifs = notifications.filter(n => !n.read).length;

  const tabs = [
    { key: "home", screen: SCREEN.HOME, icon: "home", label: "Home" },
    { key: "jobs", screen: SCREEN.JOBS, icon: "briefcase", label: "Jobs" },
    { key: "earnings", screen: SCREEN.EARNINGS, icon: "wallet", label: "Earnings" },
    { key: "profile", screen: SCREEN.PROFILE, icon: "user", label: "Profile" },
  ];

  const isSubScreen = [SCREEN.JOB_DETAIL, SCREEN.NOTIFICATIONS, SCREEN.RATINGS, SCREEN.TRAINING].includes(screen);

  const sharedProps = { addToast, addApiCall, removeApiCall };

  const renderScreen = () => {
    switch (screen) {
      case SCREEN.LOGIN:
        return <LoginScreen onLogin={handleLogin} {...sharedProps} />;
      case SCREEN.HOME:
        return <HomeScreen setScreen={setScreen} setSelectedJob={setSelectedJob} jobs={jobs} partner={partner} isOnline={isOnline} toggleOnline={() => setIsOnline(v => !v)} {...sharedProps} />;
      case SCREEN.JOBS:
        return <JobsScreen setScreen={setScreen} setSelectedJob={setSelectedJob} jobs={jobs} />;
      case SCREEN.JOB_DETAIL:
        return <JobDetailScreen job={selectedJob} setScreen={setScreen} onJobUpdate={handleJobUpdate} {...sharedProps} />;
      case SCREEN.EARNINGS:
        return <EarningsScreen />;
      case SCREEN.NOTIFICATIONS:
        return <NotificationsScreen notifications={notifications} markAllRead={markAllRead} />;
      case SCREEN.RATINGS:
        return <RatingsScreen />;
      case SCREEN.TRAINING:
        return <TrainingScreen />;
      case SCREEN.PROFILE:
        return <ProfileScreen partner={partner} onLogout={handleLogout} {...sharedProps} />;
      default:
        return null;
    }
  };

  return (
    <>
      <style>{`
        @keyframes spin { to { transform: rotate(360deg); } }
        @keyframes slideDown { from { opacity: 0; transform: translateY(-8px); } to { opacity: 1; transform: translateY(0); } }
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { display: none; }
      `}</style>
      <div style={{ display: "flex", justifyContent: "center", alignItems: "center", minHeight: "100vh", background: "linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%)", fontFamily: "'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif" }}>
        <div style={{ width: 390, height: 844, background: C.bg, borderRadius: 44, overflow: "hidden", position: "relative", boxShadow: "0 30px 80px rgba(0,0,0,0.6), 0 0 0 1px rgba(255,255,255,0.1)", display: "flex", flexDirection: "column" }}>
          {/* Status bar */}
          <div style={{ background: "transparent", padding: "12px 24px 0", display: "flex", justifyContent: "space-between", alignItems: "center", position: "absolute", top: 0, left: 0, right: 0, zIndex: 100 }}>
            <span style={{ fontSize: 12, fontWeight: 700, color: "#fff" }}>9:41</span>
            <div style={{ width: 120, height: 24, background: "rgba(0,0,0,0.8)", borderRadius: 12 }} />
            <div style={{ display: "flex", gap: 4, alignItems: "center" }}>
              <span style={{ fontSize: 10, fontWeight: 700, color: "#fff" }}>●●●</span>
            </div>
          </div>

          {/* Toast container */}
          <Toast toasts={toasts} removeToast={removeToast} />

          {/* API call indicator */}
          <ApiIndicator calls={apiCalls} />

          {/* Notification bell */}
          {loggedIn && !isSubScreen && (
            <button onClick={() => setScreen(SCREEN.NOTIFICATIONS)}
              style={{ position: "absolute", top: 48, right: 16, zIndex: 99, background: "rgba(255,255,255,0.2)", border: "none", borderRadius: 12, width: 38, height: 38, display: "flex", alignItems: "center", justifyContent: "center", cursor: "pointer", backdropFilter: "blur(8px)" }}>
              <Icon name="bell" size={18} color="#fff" />
              {unreadNotifs > 0 && (
                <div style={{ position: "absolute", top: -4, right: -4, width: 16, height: 16, background: C.danger, borderRadius: "50%", border: "2px solid white", display: "flex", alignItems: "center", justifyContent: "center" }}>
                  <span style={{ fontSize: 9, color: "#fff", fontWeight: 800 }}>{unreadNotifs}</span>
                </div>
              )}
            </button>
          )}

          {/* Main content */}
          <div style={{ flex: 1, overflowY: "auto", scrollbarWidth: "none" }}>
            {renderScreen()}
          </div>

          {/* Bottom nav */}
          {loggedIn && !isSubScreen && (
            <div style={{ background: "#fff", borderTop: `1px solid ${C.border}`, padding: "8px 0 20px", display: "flex", justifyContent: "space-around", boxShadow: "0 -4px 20px rgba(0,0,0,0.08)" }}>
              {tabs.map(tab => {
                const isActive = activeTab === tab.key;
                const newCount = tab.key === "jobs" ? jobs.filter(j => j.status === "new").length : 0;
                return (
                  <button key={tab.key} onClick={() => { setActiveTab(tab.key); setScreen(tab.screen); }}
                    style={{ display: "flex", flexDirection: "column", alignItems: "center", gap: 4, background: "none", border: "none", cursor: "pointer", padding: "4px 16px", position: "relative" }}>
                    {isActive && <div style={{ position: "absolute", top: -8, left: "50%", transform: "translateX(-50%)", width: 32, height: 3, background: C.saffron, borderRadius: "0 0 4px 4px" }} />}
                    <div style={{ position: "relative" }}>
                      <Icon name={tab.icon} size={22} color={isActive ? C.saffron : C.textMuted} />
                      {newCount > 0 && (
                        <div style={{ position: "absolute", top: -4, right: -6, width: 14, height: 14, background: C.danger, borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center" }}>
                          <span style={{ fontSize: 8, color: "#fff", fontWeight: 800 }}>{newCount}</span>
                        </div>
                      )}
                    </div>
                    <span style={{ fontSize: 10, fontWeight: isActive ? 700 : 500, color: isActive ? C.saffron : C.textMuted }}>{tab.label}</span>
                  </button>
                );
              })}
            </div>
          )}
        </div>

        {/* Label + instructions */}
        <div style={{ position: "absolute", bottom: 20, textAlign: "center" }}>
          <div style={{ color: "rgba(255,255,255,0.5)", fontSize: 11, fontWeight: 600, letterSpacing: 1 }}>
            MAHAMAINTENANCEPRO PARTNER · DEMO: any 10-digit number + OTP 1234
          </div>
        </div>
      </div>
    </>
  );
}
