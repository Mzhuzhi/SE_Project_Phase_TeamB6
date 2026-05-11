# Rei Greca — Application Form

> Personal contribution to UniGate Team B6 — Phase III: Software Design

---

## What this component does

This module is the **application form** — the part of UniGate where a logged-in student actually applies to a programme. It is the bridge between the programme catalogue (where the student picks what to apply to) and the database (where the application is stored for the admin staff to review).

**In one sentence:** a three-step form that collects personal and academic information, validates it on both the frontend and the backend, and stores it in the `applications` table linked to the student and the chosen programme.

**Key design choices:**

- **Three-step flow** (personal details → academic history → review) to reduce drop-offs and make a long form feel manageable.
- **Adaptive form** — if the programme is a Master's, an extra Bachelor degree subsection appears in step 2.
- **Double validation** — the frontend validates each step for fast feedback, and the backend validates again because frontend checks can be bypassed.
- **One application per university rule** — a student cannot apply twice to the same university, enforced on both page load and at submission.

---

## Where the code lives

| Layer | File | Purpose |
|-------|------|---------|
| Frontend | `frontend/src/pages/Apply.jsx` | The entire three-step form |
| Backend | `backend/routes/applications.js` | `POST /api/applications` endpoint + server-side validation |
| Database | `backend/models/Application.js` | Sequelize model for the `applications` table |

---

### Frontend — `frontend/src/pages/Apply.jsx`

The entire multi-step form. Holds form state, renders one of three step components, runs validation before letting the user advance, handles loading / not-found / already-applied states, and calls `submitApplication()` at the end.

```jsx
import { useState, useEffect } from 'react'
import { useParams, useNavigate, Link } from 'react-router-dom'
import { useTranslation } from 'react-i18next'
import { useAuth } from '../contexts/AuthContext'
import { getProgram, getMyApplications, submitApplication } from '../lib/api'
import Navbar from '../components/Navbar'
import InfoTooltip from '../components/InfoTooltip'

// ── tiny UI helpers ──────────────────────────────────────────────────────────
function WarningIcon() {
  return (
    <svg className="w-12 h-12 text-accent mx-auto mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={1.5}>
      <path strokeLinecap="round" strokeLinejoin="round" d="M12 9v3.75m-9.303 3.376c-.866 1.5.217 3.374 1.948 3.374h14.71c1.73 0 2.813-1.874 1.948-3.374L13.949 3.378c-.866-1.5-3.032-1.5-3.898 0L2.697 16.126ZM12 15.75h.007v.008H12v-.008Z" />
    </svg>
  )
}

function CheckIcon() {
  return (
    <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2.5}>
      <path strokeLinecap="round" strokeLinejoin="round" d="m4.5 12.75 6 6 9-13.5" />
    </svg>
  )
}

function FieldError({ msg }) {
  if (!msg) return null
  return <p className="mt-1 text-sm text-red-600">{msg}</p>
}

function ButtonSpinner() {
  return (
    <svg className="animate-spin w-4 h-4 mr-2 inline-block" fill="none" viewBox="0 0 24 24">
      <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
      <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
    </svg>
  )
}

function Field({ label, error, children, tooltip }) {
  return (
    <div>
      <div className="flex items-center gap-1.5 mb-1">
        <label className="text-sm font-medium text-text">{label}</label>
        {tooltip && <InfoTooltip text={tooltip} />}
      </div>
      {children}
      <FieldError msg={error} />
    </div>
  )
}

const inputCls = (err) =>
  `w-full px-3 py-2 rounded-lg border text-sm bg-surface text-text placeholder:text-text-muted focus:outline-none focus:ring-2 focus:ring-brand/30 transition ${
    err ? 'border-red-400' : 'border-border focus:border-brand'
  }`

const textareaCls = (err) =>
  `w-full px-3 py-2 rounded-lg border text-sm bg-surface text-text placeholder:text-text-muted focus:outline-none focus:ring-2 focus:ring-brand/30 transition resize-none ${
    err ? 'border-red-400' : 'border-border focus:border-brand'
  }`

function SubsectionHeading({ children }) {
  return (
    <h3 className="text-xs font-semibold uppercase tracking-wider text-text-muted">
      {children}
    </h3>
  )
}

// ── ProgrammeSummaryCard ─────────────────────────────────────────────────────
function ProgrammeSummaryCard({ programme, t }) {
  if (!programme) return null
  return (
    <div className="bg-brand/5 border border-brand/20 rounded-xl p-4 mb-8">
      <p className="text-xs font-semibold uppercase tracking-widest text-brand mb-1">
        {t('apply.review.programme')}
      </p>
      <h2 className="font-sora font-semibold text-text text-lg leading-snug">{programme.title}</h2>
      <p className="text-sm text-text-muted mt-0.5">{programme.university_name}</p>
      <div className="flex flex-wrap gap-3 mt-3 text-xs text-text-muted">
        <span className="bg-surface border border-border rounded-full px-3 py-1">{programme.degree_type}</span>
        <span className="bg-surface border border-border rounded-full px-3 py-1">{programme.duration}</span>
        {programme.tuition_fee && (
          <span className="bg-surface border border-border rounded-full px-3 py-1">€{Number(programme.tuition_fee).toLocaleString()}</span>
        )}
      </div>
    </div>
  )
}

// ── StepIndicator ────────────────────────────────────────────────────────────
function StepIndicator({ step, t }) {
  const steps = [
    t('apply.steps.personal'),
    t('apply.steps.academic'),
    t('apply.steps.review'),
  ]
  return (
    <div className="flex items-center mb-10">
      {steps.map((label, i) => {
        const num = i + 1
        const done = num < step
        const active = num === step
        return (
          <div key={num} className="flex items-center flex-1 last:flex-none">
            <div className="flex flex-col items-center">
              <div
                className={`w-8 h-8 rounded-full flex items-center justify-center text-sm font-semibold transition-colors ${
                  done
                    ? 'bg-brand text-white'
                    : active
                    ? 'bg-brand text-white ring-4 ring-brand/20'
                    : 'bg-border text-text-muted'
                }`}
              >
                {done ? <CheckIcon /> : num}
              </div>
              <span
                className={`mt-1.5 text-xs font-medium whitespace-nowrap ${
                  active ? 'text-brand' : 'text-text-muted'
                }`}
              >
                {label}
              </span>
            </div>
            {i < steps.length - 1 && (
              <div
                className={`flex-1 h-0.5 mx-2 mb-5 transition-colors ${
                  done ? 'bg-brand' : 'bg-border'
                }`}
              />
            )}
          </div>
        )
      })}
    </div>
  )
}

// ── Step1Fields ──────────────────────────────────────────────────────────────
function Step1Fields({ fields, errors, onChange, t }) {
  return (
    <div className="space-y-5">
      <Field label={t('apply.fields.date_of_birth')} error={errors.date_of_birth} tooltip={t('apply.tooltips.date_of_birth')}>
        <input
          type="date"
          value={fields.date_of_birth}
          onChange={(e) => onChange('date_of_birth', e.target.value)}
          className={inputCls(errors.date_of_birth)}
        />
      </Field>
      <Field label={t('apply.fields.nationality')} error={errors.nationality}>
        <input
          type="text"
          value={fields.nationality}
          onChange={(e) => onChange('nationality', e.target.value)}
          placeholder={t('apply.placeholders.nationality')}
          className={inputCls(errors.nationality)}
        />
      </Field>
      <Field label={t('apply.fields.phone')} error={errors.phone} tooltip={t('apply.tooltips.phone')}>
        <input
          type="tel"
          value={fields.phone}
          onChange={(e) => onChange('phone', e.target.value)}
          placeholder={t('apply.placeholders.phone')}
          className={inputCls(errors.phone)}
        />
      </Field>
      <Field label={t('apply.fields.address')} error={errors.address}>
        <textarea
          rows={3}
          value={fields.address}
          onChange={(e) => onChange('address', e.target.value)}
          placeholder={t('apply.placeholders.address')}
          className={textareaCls(errors.address)}
        />
      </Field>
    </div>
  )
}

// ── Step2Fields ──────────────────────────────────────────────────────────────
function Step2Fields({ fields, errors, onChange, t, isMaster }) {
  const maxYear = new Date().getFullYear() + 1
  return (
    <div>
      {/* High school subsection */}
      <SubsectionHeading>{t('apply.sections.high_school')}</SubsectionHeading>
      <div className="space-y-5 mt-4">
        <Field label={t('apply.fields.high_school_name')} error={errors.high_school_name}>
          <input
            type="text"
            value={fields.high_school_name}
            onChange={(e) => onChange('high_school_name', e.target.value)}
            placeholder={t('apply.placeholders.high_school_name')}
            className={inputCls(errors.high_school_name)}
          />
        </Field>
        <Field label={t('apply.fields.high_school_country')} error={errors.high_school_country}>
          <input
            type="text"
            value={fields.high_school_country}
            onChange={(e) => onChange('high_school_country', e.target.value)}
            placeholder={t('apply.placeholders.high_school_country')}
            className={inputCls(errors.high_school_country)}
          />
        </Field>
        <Field label={t('apply.fields.graduation_year')} error={errors.graduation_year} tooltip={t('apply.tooltips.graduation_year')}>
          <input
            type="number"
            value={fields.graduation_year}
            onChange={(e) => onChange('graduation_year', e.target.value)}
            placeholder={t('apply.placeholders.graduation_year')}
            min="1950"
            max={maxYear}
            className={inputCls(errors.graduation_year)}
          />
        </Field>
        <Field label={t('apply.fields.gpa')} error={errors.gpa} tooltip={t('apply.tooltips.gpa')}>
          <input
            type="number"
            value={fields.gpa}
            onChange={(e) => onChange('gpa', e.target.value)}
            placeholder={t('apply.placeholders.gpa')}
            min="0"
            max="4"
            step="0.01"
            className={inputCls(errors.gpa)}
          />
        </Field>
      </div>

      {/* Bachelor degree subsection — Master applicants only */}
      {isMaster && (
        <>
          <div className="border-t border-border my-6" />
          <SubsectionHeading>{t('apply.sections.bachelor')}</SubsectionHeading>
          <div className="space-y-5 mt-4">
            <Field label={t('apply.fields.bachelor_school_name')} error={errors.bachelor_school_name}>
              <input
                type="text"
                value={fields.bachelor_school_name}
                onChange={(e) => onChange('bachelor_school_name', e.target.value)}
                placeholder={t('apply.placeholders.bachelor_school_name')}
                className={inputCls(errors.bachelor_school_name)}
              />
            </Field>
            <Field label={t('apply.fields.bachelor_school_country')} error={errors.bachelor_school_country}>
              <input
                type="text"
                value={fields.bachelor_school_country}
                onChange={(e) => onChange('bachelor_school_country', e.target.value)}
                placeholder={t('apply.placeholders.bachelor_school_country')}
                className={inputCls(errors.bachelor_school_country)}
              />
            </Field>
            <Field label={t('apply.fields.bachelor_graduation_year')} error={errors.bachelor_graduation_year} tooltip={t('apply.tooltips.bachelor_graduation_year')}>
              <input
                type="number"
                value={fields.bachelor_graduation_year}
                onChange={(e) => onChange('bachelor_graduation_year', e.target.value)}
                placeholder={t('apply.placeholders.bachelor_graduation_year')}
                min="1950"
                max={maxYear}
                className={inputCls(errors.bachelor_graduation_year)}
              />
            </Field>
            <Field label={t('apply.fields.bachelor_gpa')} error={errors.bachelor_gpa} tooltip={t('apply.tooltips.bachelor_gpa')}>
              <input
                type="number"
                value={fields.bachelor_gpa}
                onChange={(e) => onChange('bachelor_gpa', e.target.value)}
                placeholder={t('apply.placeholders.bachelor_gpa')}
                min="0"
                max="4"
                step="0.01"
                className={inputCls(errors.bachelor_gpa)}
              />
            </Field>
          </div>
        </>
      )}

      {/* Additional notes — always below all subsections */}
      <div className="mt-5">
        <Field label={t('apply.fields.additional_notes')} error={errors.additional_notes}>
          <textarea
            rows={4}
            value={fields.additional_notes}
            onChange={(e) => onChange('additional_notes', e.target.value)}
            placeholder={t('apply.placeholders.additional_notes')}
            className={textareaCls(errors.additional_notes)}
          />
        </Field>
      </div>
    </div>
  )
}

// ── Step3Review ──────────────────────────────────────────────────────────────
function ReviewRow({ label, value }) {
  if (!value && value !== 0) return null
  return (
    <div className="flex gap-4 py-3 border-b border-border last:border-0">
      <dt className="w-40 flex-shrink-0 text-sm text-text-muted">{label}</dt>
      <dd className="text-sm text-text">{value}</dd>
    </div>
  )
}

function Step3Review({ fields, programme, t }) {
  const isMaster = programme?.degree_type === 'Master'
  return (
    <div className="space-y-6">
      {/* Personal details */}
      <div>
        <h3 className="text-sm font-semibold uppercase tracking-widest text-brand mb-3">
          {t('apply.review.personal')}
        </h3>
        <dl className="bg-surface border border-border rounded-xl px-4">
          <ReviewRow label={t('apply.fields.date_of_birth')} value={fields.date_of_birth} />
          <ReviewRow label={t('apply.fields.nationality')} value={fields.nationality} />
          <ReviewRow label={t('apply.fields.phone')} value={fields.phone} />
          <ReviewRow label={t('apply.fields.address')} value={fields.address} />
        </dl>
      </div>

      {/* Academic history */}
      <div>
        <h3 className="text-sm font-semibold uppercase tracking-widest text-brand mb-3">
          {t('apply.review.academic')}
        </h3>
        {isMaster ? (
          // Master: two labelled subsection blocks
          <div className="space-y-3">
            <p className="text-xs font-semibold uppercase tracking-wider text-text-muted">
              {t('apply.sections.high_school')}
            </p>
            <dl className="bg-surface border border-border rounded-xl px-4">
              <ReviewRow label={t('apply.fields.high_school_name')} value={fields.high_school_name} />
              <ReviewRow label={t('apply.fields.high_school_country')} value={fields.high_school_country} />
              <ReviewRow label={t('apply.fields.graduation_year')} value={fields.graduation_year} />
              {fields.gpa && <ReviewRow label={t('apply.fields.gpa')} value={fields.gpa} />}
            </dl>
            <p className="text-xs font-semibold uppercase tracking-wider text-text-muted !mt-4">
              {t('apply.sections.bachelor')}
            </p>
            <dl className="bg-surface border border-border rounded-xl px-4">
              <ReviewRow label={t('apply.fields.bachelor_school_name')} value={fields.bachelor_school_name} />
              <ReviewRow label={t('apply.fields.bachelor_school_country')} value={fields.bachelor_school_country} />
              <ReviewRow label={t('apply.fields.bachelor_graduation_year')} value={fields.bachelor_graduation_year} />
              {fields.bachelor_gpa && (
                <ReviewRow label={t('apply.fields.bachelor_gpa')} value={fields.bachelor_gpa} />
              )}
            </dl>
            {fields.additional_notes && (
              <dl className="bg-surface border border-border rounded-xl px-4 !mt-3">
                <ReviewRow label={t('apply.fields.additional_notes')} value={fields.additional_notes} />
              </dl>
            )}
          </div>
        ) : (
          // Bachelor: single block, same as before
          <dl className="bg-surface border border-border rounded-xl px-4">
            <ReviewRow label={t('apply.fields.high_school_name')} value={fields.high_school_name} />
            <ReviewRow label={t('apply.fields.high_school_country')} value={fields.high_school_country} />
            <ReviewRow label={t('apply.fields.graduation_year')} value={fields.graduation_year} />
            {fields.gpa && <ReviewRow label={t('apply.fields.gpa')} value={fields.gpa} />}
            {fields.additional_notes && (
              <ReviewRow label={t('apply.fields.additional_notes')} value={fields.additional_notes} />
            )}
          </dl>
        )}
      </div>
    </div>
  )
}

// ── Validation ───────────────────────────────────────────────────────────────
function validateStep1(fields, t) {
  const errs = {}
  if (!fields.date_of_birth?.trim()) errs.date_of_birth = t('apply.validation.date_of_birth_required')
  if (!fields.nationality?.trim()) errs.nationality = t('apply.validation.nationality_required')
  if (!fields.phone?.trim()) errs.phone = t('apply.validation.phone_required')
  if (!fields.address?.trim()) errs.address = t('apply.validation.address_required')
  return errs
}

function validateStep2(fields, t, isMaster = false) {
  const errs = {}
  const maxYear = new Date().getFullYear() + 1

  // High school — required for all applicants
  if (!fields.high_school_name?.trim()) errs.high_school_name = t('apply.validation.high_school_name_required')
  if (!fields.high_school_country?.trim()) errs.high_school_country = t('apply.validation.high_school_country_required')
  if (!fields.graduation_year?.toString().trim()) {
    errs.graduation_year = t('apply.validation.graduation_year_required')
  } else {
    const yr = parseInt(fields.graduation_year, 10)
    if (isNaN(yr) || yr < 1950 || yr > maxYear) {
      errs.graduation_year = t('apply.validation.graduation_year_invalid')
    }
  }
  const gpaRaw = fields.gpa?.toString().trim()
  if (!gpaRaw) {
    errs.gpa = t('apply.validation.gpa_required')
  } else {
    const n = parseFloat(gpaRaw)
    if (isNaN(n) || n < 0 || n > 4) errs.gpa = t('apply.validation.gpa_range')
  }

  // Bachelor degree — required only for Master programmes
  if (isMaster) {
    if (!fields.bachelor_school_name?.trim()) {
      errs.bachelor_school_name = t('apply.validation.bachelor_school_name_required')
    }
    if (!fields.bachelor_school_country?.trim()) {
      errs.bachelor_school_country = t('apply.validation.bachelor_school_country_required')
    }
    if (!fields.bachelor_graduation_year?.toString().trim()) {
      errs.bachelor_graduation_year = t('apply.validation.bachelor_graduation_year_required')
    } else {
      const byr = parseInt(fields.bachelor_graduation_year, 10)
      if (isNaN(byr) || byr < 1950 || byr > maxYear) {
        errs.bachelor_graduation_year = t('apply.validation.bachelor_graduation_year_invalid', { max: maxYear })
      }
    }
    const bgpaRaw = fields.bachelor_gpa?.toString().trim()
    if (!bgpaRaw) {
      errs.bachelor_gpa = t('apply.validation.bachelor_gpa_required')
    } else {
      const bn = parseFloat(bgpaRaw)
      if (isNaN(bn) || bn < 0 || bn > 4) errs.bachelor_gpa = t('apply.validation.bachelor_gpa_range')
    }
  }

  return errs
}

// ── Apply page ───────────────────────────────────────────────────────────────
export default function Apply() {
  const { programmeId } = useParams()
  const navigate = useNavigate()
  const { t } = useTranslation()
  const { user } = useAuth()

  const [programme, setProgramme] = useState(null)
  const [pageLoading, setPageLoading] = useState(true)
  const [notFound, setNotFound] = useState(false)
  const [alreadyApplied, setAlreadyApplied] = useState(false)
  const [step, setStep] = useState(1)
  const [fadeIn, setFadeIn] = useState(true)
  const [fields, setFields] = useState({
    date_of_birth: '',
    nationality: '',
    phone: '',
    address: '',
    high_school_name: '',
    high_school_country: '',
    graduation_year: '',
    gpa: '',
    additional_notes: '',
    bachelor_school_name: '',
    bachelor_school_country: '',
    bachelor_graduation_year: '',
    bachelor_gpa: '',
  })
  const [errors, setErrors] = useState({})
  const [attempted, setAttempted] = useState(false)
  const [submitting, setSubmitting] = useState(false)
  const [submitError, setSubmitError] = useState(null)

  const isMaster = programme?.degree_type === 'Master'

  // Role guard — only students may apply
  useEffect(() => {
    if (user && user.role !== 'student') navigate('/', { replace: true })
  }, [user, navigate])

  // Load programme + check for existing application at same university
  useEffect(() => {
    let cancelled = false
    async function load() {
      try {
        const [progRes, appsRes] = await Promise.all([
          getProgram(programmeId),
          getMyApplications().catch(() => ({ data: [] })),
        ])
        if (cancelled) return
        const prog = progRes.data
        const apps = appsRes.data ?? []
        const conflict = apps.some(
          (a) => a.programme?.university_id === prog.university_id
        )
        setProgramme(prog)
        setAlreadyApplied(conflict)
      } catch (err) {
        if (!cancelled) setNotFound(true)
      } finally {
        if (!cancelled) setPageLoading(false)
      }
    }
    load()
    return () => { cancelled = true }
  }, [programmeId])

  function handleChange(name, value) {
    setFields((prev) => ({ ...prev, [name]: value }))
    if (attempted) {
      const next = { ...fields, [name]: value }
      setErrors(
        step === 1
          ? validateStep1(next, t)
          : validateStep2(next, t, isMaster)
      )
    }
  }

  function transition(toStep) {
    setFadeIn(false)
    setTimeout(() => {
      setStep(toStep)
      setErrors({})
      setAttempted(false)
      setFadeIn(true)
    }, 150)
  }

  function handleNext() {
    const errs = step === 1
      ? validateStep1(fields, t)
      : validateStep2(fields, t, isMaster)
    if (Object.keys(errs).length > 0) {
      setErrors(errs)
      setAttempted(true)
      return
    }
    transition(step + 1)
  }

  function handleBack() {
    transition(step - 1)
  }

  async function handleSubmit() {
    setSubmitting(true)
    setSubmitError(null)
    try {
      const payload = {
        program_id: programmeId,
        date_of_birth: fields.date_of_birth,
        nationality: fields.nationality,
        phone: fields.phone,
        address: fields.address,
        high_school_name: fields.high_school_name,
        high_school_country: fields.high_school_country,
        graduation_year: fields.graduation_year,
        gpa: fields.gpa,
        additional_notes: fields.additional_notes || undefined,
      }
      if (isMaster) {
        payload.bachelor_school_name = fields.bachelor_school_name
        payload.bachelor_school_country = fields.bachelor_school_country
        payload.bachelor_graduation_year = fields.bachelor_graduation_year
        payload.bachelor_gpa = fields.bachelor_gpa
      }
      await submitApplication(payload)
      navigate('/dashboard', { state: { programmeName: programme?.title } })
    } catch {
      setSubmitError(t('apply.errors.submit_failed'))
      setSubmitting(false)
    }
  }

  // ── loading ──
  if (pageLoading) {
    return (
      <>
        <Navbar />
        <div className="min-h-screen bg-bg flex items-center justify-center">
          <p className="text-text-muted">{t('apply.loading')}</p>
        </div>
      </>
    )
  }

  // ── not found ──
  if (notFound) {
    return (
      <>
        <Navbar />
        <div className="min-h-screen bg-bg flex items-center justify-center px-4">
          <div className="text-center max-w-sm">
            <WarningIcon />
            <h1 className="font-sora font-bold text-xl text-text mb-2">{t('apply.not_found.title')}</h1>
            <p className="text-text-muted text-sm mb-6">{t('apply.not_found.subtitle')}</p>
            <Link to="/" className="text-brand font-medium text-sm hover:underline">
              ← {t('apply.not_found.back')}
            </Link>
          </div>
        </div>
      </>
    )
  }

  // ── already applied ──
  if (alreadyApplied) {
    return (
      <>
        <Navbar />
        <div className="min-h-screen bg-bg flex items-center justify-center px-4">
          <div className="text-center max-w-sm">
            <WarningIcon />
            <h1 className="font-sora font-bold text-xl text-text mb-2">{t('apply.already_applied.title')}</h1>
            <p className="text-text-muted text-sm mb-6">{t('apply.already_applied.subtitle')}</p>
            <Link to="/" className="text-brand font-medium text-sm hover:underline">
              ← {t('apply.already_applied.back')}
            </Link>
          </div>
        </div>
      </>
    )
  }

  // ── main form ──
  return (
    <>
      <Navbar />
      <div className="min-h-screen bg-bg py-12 px-4">
        <div className="max-w-xl mx-auto">
          <h1 className="font-sora font-bold text-2xl text-text mb-8">
            {t('apply.title')} <span className="text-brand">{programme?.university_name}</span>
          </h1>
          <ProgrammeSummaryCard programme={programme} t={t} />
          <StepIndicator step={step} t={t} />
          <div
            className="transition-opacity duration-150"
            style={{ opacity: fadeIn ? 1 : 0 }}
          >
            {step === 1 && (
              <Step1Fields fields={fields} errors={errors} onChange={handleChange} t={t} />
            )}
            {step === 2 && (
              <Step2Fields fields={fields} errors={errors} onChange={handleChange} t={t} isMaster={isMaster} />
            )}
            {step === 3 && (
              <>
                <p className="text-sm text-text-muted mb-6">{t('apply.review.subtitle')}</p>
                <Step3Review fields={fields} programme={programme} t={t} />
              </>
            )}
            {submitError && (
              <p className="mt-6 text-sm text-red-600 text-center">{submitError}</p>
            )}
            <div className="flex gap-3 mt-8">
              {step > 1 && (
                <button
                  type="button"
                  onClick={handleBack}
                  disabled={submitting}
                  className="flex-1 py-2.5 rounded-lg border border-border text-sm font-medium text-text hover:bg-border/40 transition disabled:opacity-50"
                >
                  {t('apply.buttons.back')}
                </button>
              )}
              {step < 3 ? (
                <button
                  type="button"
                  onClick={handleNext}
                  className="flex-1 py-2.5 rounded-lg bg-brand text-white text-sm font-semibold hover:bg-brand/90 transition"
                >
                  {t('apply.buttons.next')}
                </button>
              ) : (
                <button
                  type="button"
                  onClick={handleSubmit}
                  disabled={submitting}
                  className="flex-1 py-2.5 rounded-lg bg-accent text-white text-sm font-semibold hover:bg-accent/90 transition disabled:opacity-70"
                >
                  {submitting && <ButtonSpinner />}
                  {submitting ? t('apply.buttons.submitting') : t('apply.buttons.submit')}
                </button>
              )}
            </div>
          </div>
        </div>
      </div>
    </>
  )
}
```

---

### Backend — `backend/routes/applications.js`

The `POST /api/applications` endpoint with all server-side validation, plus `GET /api/applications/mine` for the student dashboard and `GET /api/applications/:id` for the admin view.

```js
const express = require('express');
const { Application, Program, University, User, Document } = require('../models');
const requireAuth = require('../middleware/requireAuth');
const requireRole = require('../middleware/requireRole');

const router = express.Router();

const CURRENT_YEAR = () => new Date().getFullYear();

// Shapes a Sequelize Program instance into a clean response object,
// choosing the right language fields based on the student's preference.
function pickProgramLang(program, lang) {
  const l = lang === 'sq' ? 'sq' : 'en';
  const p = program.toJSON ? program.toJSON() : program;
  return {
    id: p.id,
    university_id: p.university_id,
    university_name: p.University?.name ?? null,
    degree_type: p.degree_type,
    duration: p.duration,
    language_of_instruction: p.language_of_instruction,
    tuition_fee: p.tuition_fee,
    title: p[`title_${l}`],
    required_document_types: p.required_document_types ?? [],
  };
}

// Returns the student's stored language preference ('en' or 'sq').
// Falls back to 'en' if the user row is missing for any reason.
async function getUserLang(userId) {
  const user = await User.findByPk(userId, { attributes: ['language_preference'] });
  return user?.language_preference ?? 'en';
}

// POST /api/applications
router.post('/', requireAuth, requireRole('student'), async (req, res) => {
  try {
    const {
      program_id,
      date_of_birth,
      nationality,
      phone,
      address,
      high_school_name,
      high_school_country,
      graduation_year,
      gpa,
      additional_notes,
      bachelor_school_name,
      bachelor_school_country,
      bachelor_graduation_year,
      bachelor_gpa,
    } = req.body;

    // --- Required field validation ---
    const required = {
      program_id,
      date_of_birth,
      nationality,
      phone,
      address,
      high_school_name,
      high_school_country,
      graduation_year,
    };
    for (const [field, value] of Object.entries(required)) {
      if (value === undefined || value === null || String(value).trim() === '') {
        return res.status(400).json({
          status: 'error',
          message: `${field} is required`,
        });
      }
    }

    // --- graduation_year range ---
    const yearInt = parseInt(graduation_year, 10);
    if (isNaN(yearInt) || yearInt < 1950 || yearInt > CURRENT_YEAR() + 1) {
      return res.status(400).json({
        status: 'error',
        message: `graduation_year must be a year between 1950 and ${CURRENT_YEAR() + 1}`,
      });
    }

    // --- GPA (required) ---
    if (gpa === undefined || gpa === null || String(gpa).trim() === '') {
      return res.status(400).json({ status: 'error', message: 'gpa is required' });
    }
    const gpaValue = parseFloat(gpa);
    if (isNaN(gpaValue) || gpaValue < 0 || gpaValue > 4) {
      return res.status(400).json({
        status: 'error',
        message: 'gpa must be a number between 0 and 4',
      });
    }

    // --- Programme existence check ---
    const programme = await Program.findByPk(program_id, {
      include: [{ model: University, attributes: ['id', 'name'] }],
    });
    if (!programme) {
      return res.status(404).json({
        status: 'error',
        message: 'Programme not found',
      });
    }

    // --- Master-specific: validate bachelor degree info ---
    let bachelorSchoolName = null;
    let bachelorSchoolCountry = null;
    let bachelorGraduationYear = null;
    let bachelorGpaValue = null;

    if (programme.degree_type === 'Master') {
      if (!bachelor_school_name || String(bachelor_school_name).trim() === '') {
        return res.status(400).json({ status: 'error', message: 'bachelor_school_name is required for Master programmes' });
      }
      if (!bachelor_school_country || String(bachelor_school_country).trim() === '') {
        return res.status(400).json({ status: 'error', message: 'bachelor_school_country is required for Master programmes' });
      }
      if (bachelor_graduation_year === undefined || bachelor_graduation_year === null || String(bachelor_graduation_year).trim() === '') {
        return res.status(400).json({ status: 'error', message: 'bachelor_graduation_year is required for Master programmes' });
      }
      const bachelorYearInt = parseInt(bachelor_graduation_year, 10);
      if (isNaN(bachelorYearInt) || bachelorYearInt < 1950 || bachelorYearInt > CURRENT_YEAR() + 1) {
        return res.status(400).json({
          status: 'error',
          message: `bachelor_graduation_year must be a year between 1950 and ${CURRENT_YEAR() + 1}`,
        });
      }
      bachelorSchoolName = String(bachelor_school_name).trim();
      bachelorSchoolCountry = String(bachelor_school_country).trim();
      bachelorGraduationYear = bachelorYearInt;

      if (bachelor_gpa === undefined || bachelor_gpa === null || String(bachelor_gpa).trim() === '') {
        return res.status(400).json({ status: 'error', message: 'bachelor_gpa is required for Master programmes' });
      }
      bachelorGpaValue = parseFloat(bachelor_gpa);
      if (isNaN(bachelorGpaValue) || bachelorGpaValue < 0 || bachelorGpaValue > 4) {
        return res.status(400).json({ status: 'error', message: 'bachelor_gpa must be a number between 0 and 4' });
      }
    }
    // If Bachelor programme, bachelor_* fields stay null regardless of what was sent.

    // --- One application per university rule ---
    // Check if this student already has an application whose programme
    // belongs to the same university as the requested programme.
    const conflict = await Application.findOne({
      where: { user_id: req.user.id },
      include: [{
        model: Program,
        where: { university_id: programme.university_id },
        required: true,
        attributes: [],
      }],
    });
    if (conflict) {
      return res.status(409).json({
        status: 'error',
        message: `You have already applied to a programme at ${programme.University.name}. Only one application per university is allowed.`,
      });
    }

    // --- Create the application ---
    const application = await Application.create({
      user_id: req.user.id,
      program_id,
      status: 'pending',
      submitted_at: new Date(),
      date_of_birth: String(date_of_birth).trim(),
      nationality: String(nationality).trim(),
      phone: String(phone).trim(),
      address: String(address).trim(),
      high_school_name: String(high_school_name).trim(),
      high_school_country: String(high_school_country).trim(),
      graduation_year: yearInt,
      gpa: gpaValue,
      additional_notes: additional_notes ? String(additional_notes).trim() : null,
      bachelor_school_name: bachelorSchoolName,
      bachelor_school_country: bachelorSchoolCountry,
      bachelor_graduation_year: bachelorGraduationYear,
      bachelor_gpa: bachelorGpaValue,
    });

    res.status(201).json({
      status: 'ok',
      data: {
        id: application.id,
        status: application.status,
        submitted_at: application.submitted_at,
      },
    });
  } catch (err) {
    console.error(err);
    res.status(500).json({ status: 'error', message: 'Failed to submit application' });
  }
});

// GET /api/applications/mine
// Must be defined before /:id so Express matches it correctly
router.get('/mine', requireAuth, requireRole('student'), async (req, res) => {
  try {
    const lang = await getUserLang(req.user.id);
    const applications = await Application.findAll({
      where: { user_id: req.user.id },
      include: [
        {
          model: Program,
          include: [{ model: University, attributes: ['name'] }],
        },
        {
          model: Document,
          attributes: ['id', 'document_type', 'original_filename', 'size_bytes', 'uploaded_at'],
        },
      ],
      order: [['submitted_at', 'DESC']],
    });

    const data = applications.map((app) => {
      const a = app.toJSON();
      return {
        id: a.id,
        status: a.status,
        submitted_at: a.submitted_at,
        date_of_birth: a.date_of_birth,
        nationality: a.nationality,
        phone: a.phone,
        address: a.address,
        high_school_name: a.high_school_name,
        high_school_country: a.high_school_country,
        graduation_year: a.graduation_year,
        gpa: a.gpa,
        additional_notes: a.additional_notes,
        bachelor_school_name: a.bachelor_school_name,
        bachelor_school_country: a.bachelor_school_country,
        bachelor_graduation_year: a.bachelor_graduation_year,
        bachelor_gpa: a.bachelor_gpa,
        programme: pickProgramLang(app.Program, lang),
        documents: a.Documents,
      };
    });

    res.json({ status: 'ok', data });
  } catch (err) {
    console.error(err);
    res.status(500).json({ status: 'error', message: 'Failed to fetch applications' });
  }
});

// GET /api/applications/:id
router.get('/:id', requireAuth, async (req, res) => {
  try {
    const application = await Application.findByPk(req.params.id, {
      include: [
        {
          model: Program,
          include: [{ model: University, attributes: ['name', 'contact_email', 'contact_phone'] }],
        },
        {
          model: Document,
          attributes: ['id', 'document_type', 'original_filename', 'size_bytes', 'uploaded_at'],
        },
      ],
    });

    if (!application) {
      return res.status(404).json({ status: 'error', message: 'Application not found' });
    }

    // Students may only view their own applications; staff and admin can view any
    if (req.user.role === 'student' && application.user_id !== req.user.id) {
      return res.status(403).json({ status: 'error', message: 'Access denied' });
    }

    const lang = await getUserLang(req.user.id);
    const a = application.toJSON();
    res.json({
      status: 'ok',
      data: {
        id: a.id,
        status: a.status,
        submitted_at: a.submitted_at,
        date_of_birth: a.date_of_birth,
        nationality: a.nationality,
        phone: a.phone,
        address: a.address,
        high_school_name: a.high_school_name,
        high_school_country: a.high_school_country,
        graduation_year: a.graduation_year,
        gpa: a.gpa,
        additional_notes: a.additional_notes,
        bachelor_school_name: a.bachelor_school_name,
        bachelor_school_country: a.bachelor_school_country,
        bachelor_graduation_year: a.bachelor_graduation_year,
        bachelor_gpa: a.bachelor_gpa,
        programme: pickProgramLang(application.Program, lang),
        documents: a.Documents,
      },
    });
  } catch (err) {
    console.error(err);
    res.status(500).json({ status: 'error', message: 'Failed to fetch application' });
  }
});

module.exports = router;
```

---

### Database — `backend/models/Application.js`

The Sequelize model defining the `applications` table. Personal and academic fields are `NOT NULL`. The four `bachelor_*` fields are nullable because they only apply to Master applications. The `status` column is an ENUM used by the admin dashboard.

```js
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Application = sequelize.define('Application', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
  },
  user_id: {
    type: DataTypes.INTEGER,
    allowNull: false,
  },
  program_id: {
    type: DataTypes.INTEGER,
    allowNull: false,
  },
  status: {
    type: DataTypes.ENUM('pending', 'accepted', 'rejected'),
    allowNull: false,
    defaultValue: 'pending',
  },
  // Personal details
  date_of_birth: {
    type: DataTypes.DATEONLY,
    allowNull: false,
  },
  nationality: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  phone: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  address: {
    type: DataTypes.TEXT,
    allowNull: false,
  },
  // Academic history
  high_school_name: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  high_school_country: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  graduation_year: {
    type: DataTypes.INTEGER,
    allowNull: false,
  },
  gpa: {
    type: DataTypes.DECIMAL(3, 2),
    allowNull: true,
  },
  additional_notes: {
    type: DataTypes.TEXT,
    allowNull: true,
  },
  // Bachelor degree info — filled only when applying to a Master programme; null otherwise
  bachelor_school_name: {
    type: DataTypes.STRING,
    allowNull: true,
    defaultValue: null,
  },
  bachelor_school_country: {
    type: DataTypes.STRING,
    allowNull: true,
    defaultValue: null,
  },
  bachelor_graduation_year: {
    type: DataTypes.INTEGER,
    allowNull: true,
    defaultValue: null,
  },
  bachelor_gpa: {
    type: DataTypes.DECIMAL(3, 2),
    allowNull: true,
    defaultValue: null,
  },
  submitted_at: {
    type: DataTypes.DATE,
    allowNull: false,
    defaultValue: DataTypes.NOW,
  },
  // Set when a staff member moves the application from pending to accepted/rejected
  decided_at: {
    type: DataTypes.DATE,
    allowNull: true,
    defaultValue: null,
  },
}, {
  tableName: 'applications',
  timestamps: true,
});

module.exports = Application;
```
