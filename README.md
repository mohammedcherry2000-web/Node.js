// Resume Builder Application
class ResumeBuilder {
    constructor() {
        this.currentSection = 0;
        this.sections = ['personal', 'experience', 'education', 'skills'];
        this.data = this.loadData();
        this.currentTemplate = 'modern';
        this.zoomLevel = 100;
        
        this.init();
    }
    
    init() {
        this.bindEvents();
        this.populateForm();
        this.updatePreview();
        this.updateProgressIndicator();
        
        // Auto-save every 30 seconds
        setInterval(() => {
            this.saveData();
        }, 30000);
    }
    
    bindEvents() {
        // Form navigation
        document.getElementById('next-btn').addEventListener('click', () => this.nextSection());
        document.getElementById('prev-btn').addEventListener('click', () => this.prevSection());
        
        // Progress indicator clicks
        document.querySelectorAll('.progress-step').forEach((step, index) => {
            step.addEventListener('click', () => this.goToSection(index));
        });
        
        // Template selector
        document.getElementById('template-selector').addEventListener('change', (e) => {
            this.currentTemplate = e.target.value;
            this.updatePreview();
        });
        
        // Header actions
        document.getElementById('save-btn').addEventListener('click', () => this.saveData(true));
        document.getElementById('print-btn').addEventListener('click', () => this.printResume());
        document.getElementById('clear-btn').addEventListener('click', () => this.showClearConfirmation());
        
        // Experience and education management
        document.getElementById('add-experience').addEventListener('click', () => this.addExperience());
        document.getElementById('add-education').addEventListener('click', () => this.addEducation());
        
        // Zoom controls
        document.getElementById('zoom-in').addEventListener('click', () => this.zoomIn());
        document.getElementById('zoom-out').addEventListener('click', () => this.zoomOut());
        
        // Modal events
        document.getElementById('modal-close').addEventListener('click', () => this.hideModal());
        document.getElementById('modal-cancel').addEventListener('click', () => this.hideModal());
        document.getElementById('modal-confirm').addEventListener('click', () => this.confirmAction());
        
        // Form input events for live preview
        this.bindFormInputs();
        
        // Keyboard shortcuts
        document.addEventListener('keydown', (e) => this.handleKeyboardShortcuts(e));
    }
    
    bindFormInputs() {
        // Personal info inputs
        const personalInputs = [
            'full-name', 'email', 'phone', 'city', 'state', 
            'linkedin', 'website', 'summary'
        ];
        
        personalInputs.forEach(inputId => {
            const input = document.getElementById(inputId);
            if (input) {
                input.addEventListener('input', () => {
                    this.updateData();
                    this.updatePreview();
                    this.validateField(input);
                });
            }
        });
        
        // Skills inputs
        const skillsInputs = [
            'technical-skills', 'soft-skills', 'languages', 'certifications'
        ];
        
        skillsInputs.forEach(inputId => {
            const input = document.getElementById(inputId);
            if (input) {
                input.addEventListener('input', () => {
                    this.updateData();
                    this.updatePreview();
                });
            }
        });
    }
    
    // Data Management
    loadData() {
        const savedData = localStorage.getItem('resume-builder-data');
        if (savedData) {
            try {
                return JSON.parse(savedData);
            } catch (e) {
                console.error('Error parsing saved data:', e);
            }
        }
        
        return {
            fullName: '',
            email: '',
            phone: '',
            city: '',
            state: '',
            linkedin: '',
            website: '',
            summary: '',
            experience: [],
            education: [],
            technicalSkills: '',
            softSkills: '',
            languages: '',
            certifications: ''
        };
    }
    
    saveData(showToast = false) {
        try {
            localStorage.setItem('resume-builder-data', JSON.stringify(this.data));
            if (showToast) {
                this.showToast('success', 'Progress saved successfully!');
            }
        } catch (e) {
            console.error('Error saving data:', e);
            if (showToast) {
                this.showToast('error', 'Failed to save progress. Please try again.');
            }
        }
    }
    
    updateData() {
        // Update personal information
        const personalFields = [
            'fullName', 'email', 'phone', 'city', 'state', 
            'linkedin', 'website', 'summary'
        ];
        
        personalFields.forEach(field => {
            const input = document.getElementById(field.replace(/([A-Z])/g, '-$1').toLowerCase());
            if (input) {
                this.data[field] = input.value.trim();
            }
        });
        
        // Update skills
        const skillsFields = ['technicalSkills', 'softSkills', 'languages', 'certifications'];
        skillsFields.forEach(field => {
            const input = document.getElementById(field.replace(/([A-Z])/g, '-$1').toLowerCase());
            if (input) {
                this.data[field] = input.value.trim();
            }
        });
        
        // Update experience and education (handled separately)
        this.updateExperienceData();
        this.updateEducationData();
    }
    
    populateForm() {
        // Populate personal information
        const personalFields = [
            'fullName', 'email', 'phone', 'city', 'state', 
            'linkedin', 'website', 'summary'
        ];
        
        personalFields.forEach(field => {
            const input = document.getElementById(field.replace(/([A-Z])/g, '-$1').toLowerCase());
            if (input && this.data[field]) {
                input.value = this.data[field];
            }
        });
        
        // Populate skills
        const skillsFields = ['technicalSkills', 'softSkills', 'languages', 'certifications'];
        skillsFields.forEach(field => {
            const input = document.getElementById(field.replace(/([A-Z])/g, '-$1').toLowerCase());
            if (input && this.data[field]) {
                input.value = this.data[field];
            }
        });
        
        // Populate experience and education
        this.renderExperience();
        this.renderEducation();
    }
    
    // Section Navigation
    nextSection() {
        if (this.validateCurrentSection()) {
            if (this.currentSection < this.sections.length - 1) {
                this.currentSection++;
                this.showSection();
                this.updateProgressIndicator();
            }
        }
    }
    
    prevSection() {
        if (this.currentSection > 0) {
            this.currentSection--;
            this.showSection();
            this.updateProgressIndicator();
        }
    }
    
    goToSection(index) {
        if (index >= 0 && index < this.sections.length) {
            this.currentSection = index;
            this.showSection();
            this.updateProgressIndicator();
        }
    }
    
    showSection() {
        // Hide all sections
        document.querySelectorAll('.form-section').forEach(section => {
            section.classList.remove('active');
        });
        
        // Show current section
        const currentSectionElement = document.getElementById(`${this.sections[this.currentSection]}-section`);
        if (currentSectionElement) {
            currentSectionElement.classList.add('active');
        }
        
        // Update navigation buttons
        const prevBtn = document.getElementById('prev-btn');
        const nextBtn = document.getElementById('next-btn');
        
        prevBtn.style.display = this.currentSection > 0 ? 'block' : 'none';
        nextBtn.textContent = this.currentSection === this.sections.length - 1 ? 'Finish' : 'Next';
        nextBtn.innerHTML = this.currentSection === this.sections.length - 1 ? 
            'Finish <i class="fas fa-check"></i>' : 
            'Next <i class="fas fa-chevron-right"></i>';
    }
    
    updateProgressIndicator() {
        document.querySelectorAll('.progress-step').forEach((step, index) => {
            step.classList.remove('active', 'completed');
            
            if (index < this.currentSection) {
                step.classList.add('completed');
            } else if (index === this.currentSection) {
                step.classList.add('active');
            }
        });
    }
    
    // Form Validation
    validateCurrentSection() {
        const currentSectionName = this.sections[this.currentSection];
        let isValid = true;
        
        if (currentSectionName === 'personal') {
            isValid = this.validatePersonalSection();
        }
        
        return isValid;
    }
    
    validatePersonalSection() {
        let isValid = true;
        
        // Required fields
        const requiredFields = [
            { id: 'full-name', message: 'Full name is required' },
            { id: 'email', message: 'Valid email address is required' },
            { id: 'phone', message: 'Phone number is required' }
        ];
        
        requiredFields.forEach(field => {
            const input = document.getElementById(field.id);
            const errorElement = document.getElementById(`${field.id}-error`);
            
            if (!this.validateField(input)) {
                isValid = false;
                if (errorElement) {
                    errorElement.textContent = field.message;
                }
            } else if (errorElement) {
                errorElement.textContent = '';
            }
        });
        
        return isValid;
    }
    
    validateField(input) {
        if (!input) return true;
        
        const value = input.value.trim();
        
        // Check if required field is empty
        if (input.required && !value) {
            input.classList.add('error');
            return false;
        }
        
        // Email validation
        if (input.type === 'email' && value) {
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test(value)) {
                input.classList.add('error');
                return false;
            }
        }
        
        // Phone validation
        if (input.type === 'tel' && value) {
            const phoneRegex = /^[\+]?[\s\-\(\)]*([0-9][\s\-\(\)]*){10,}$/;
            if (!phoneRegex.test(value)) {
                input.classList.add('error');
                return false;
            }
        }
        
        // URL validation
        if (input.type === 'url' && value) {
            try {
                new URL(value);
            } catch {
                input.classList.add('error');
                return false;
            }
        }
        
        input.classList.remove('error');
        return true;
    }
    
    // Experience Management
    addExperience() {
        this.data.experience.push({
            title: '',
            company: '',
            startDate: '',
            endDate: '',
            current: false,
            description: ''
        });
        
        this.renderExperience();
        this.updatePreview();
        
        // Scroll to the new experience entry
        const container = document.getElementById('experience-container');
        const newEntry = container.lastElementChild;
        if (newEntry) {
            newEntry.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
            const firstInput = newEntry.querySelector('input');
            if (firstInput) firstInput.focus();
        }
    }
    
    removeExperience(index) {
        if (index >= 0 && index < this.data.experience.length) {
            this.data.experience.splice(index, 1);
            this.renderExperience();
            this.updatePreview();
            this.saveData();
        }
    }
    
    renderExperience() {
        const container = document.getElementById('experience-container');
        
        if (this.data.experience.length === 0) {
            container.innerHTML = `
                <div class="empty-state">
                    <i class="fas fa-briefcase" style="font-size: 3rem; color: var(--text-muted); margin-bottom: 1rem;"></i>
                    <p style="color: var(--text-muted); margin-bottom: 1rem;">No work experience added yet.</p>
                    <p style="color: var(--text-muted); font-size: 0.875rem;">Click "Add Experience" to get started.</p>
                </div>
            `;
            return;
        }
        
        container.innerHTML = this.data.experience.map((exp, index) => `
            <div class="experience-item">
                <div class="item-header">
                    <span class="item-number">Experience ${index + 1}</span>
                    <button type="button" class="remove-item" onclick="app.removeExperience(${index})" title="Remove this experience">
                        <i class="fas fa-trash"></i>
                    </button>
                </div>
                
                <div class="form-row">
                    <div class="form-group">
                        <label>Job Title</label>
                        <input type="text" value="${exp.title || ''}" onchange="app.updateExperienceField(${index}, 'title', this.value)">
                    </div>
                    <div class="form-group">
                        <label>Company</label>
                        <input type="text" value="${exp.company || ''}" onchange="app.updateExperienceField(${index}, 'company', this.value)">
                    </div>
                </div>
                
                <div class="form-row">
                    <div class="form-group">
                        <label>Start Date</label>
                        <input type="text" placeholder="e.g., Jan 2020" value="${exp.startDate || ''}" onchange="app.updateExperienceField(${index}, 'startDate', this.value)">
                    </div>
                    <div class="form-group">
                        <label>End Date</label>
                        <input type="text" placeholder="e.g., Dec 2022" value="${exp.endDate || ''}" onchange="app.updateExperienceField(${index}, 'endDate', this.value)" ${exp.current ? 'disabled' : ''}>
                        <div style="margin-top: 0.5rem;">
                            <label style="display: flex; align-items: center; gap: 0.5rem; font-size: 0.875rem; cursor: pointer;">
                                <input type="checkbox" ${exp.current ? 'checked' : ''} onchange="app.updateExperienceField(${index}, 'current', this.checked)">
                                Currently working here
                            </label>
                        </div>
                    </div>
                </div>
                
                <div class="form-group">
                    <label>Job Description</label>
                    <textarea rows="3" placeholder="Describe your responsibilities and achievements..." onchange="app.updateExperienceField(${index}, 'description', this.value)">${exp.description || ''}</textarea>
                </div>
            </div>
        `).join('');
    }
    
    updateExperienceField(index, field, value) {
        if (this.data.experience[index]) {
            if (field === 'current') {
                this.data.experience[index][field] = value;
                if (value) {
                    this.data.experience[index].endDate = '';
                }
                this.renderExperience(); // Re-render to update UI
            } else {
                this.data.experience[index][field] = value;
            }
            this.updatePreview();
            this.saveData();
        }
    }
    
    updateExperienceData() {
        // This method is called from updateData() but the actual updates
        // are handled by updateExperienceField() for real-time updates
    }
    
    // Education Management
    addEducation() {
        this.data.education.push({
            degree: '',
            school: '',
            graduationDate: '',
            details: ''
        });
        
        this.renderEducation();
        this.updatePreview();
        
        // Scroll to the new education entry
        const container = document.getElementById('education-container');
        const newEntry = container.lastElementChild;
        if (newEntry) {
            newEntry.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
            const firstInput = newEntry.querySelector('input');
            if (firstInput) firstInput.focus();
        }
    }
    
    removeEducation(index) {
        if (index >= 0 && index < this.data.education.length) {
            this.data.education.splice(index, 1);
            this.renderEducation();
            this.updatePreview();
            this.saveData();
        }
    }
    
    renderEducation() {
        const container = document.getElementById('education-container');
        
        if (this.data.education.length === 0) {
            container.innerHTML = `
                <div class="empty-state">
                    <i class="fas fa-graduation-cap" style="font-size: 3rem; color: var(--text-muted); margin-bottom: 1rem;"></i>
                    <p style="color: var(--text-muted); margin-bottom: 1rem;">No education entries added yet.</p>
                    <p style="color: var(--text-muted); font-size: 0.875rem;">Click "Add Education" to get started.</p>
                </div>
            `;
            return;
        }
        
        container.innerHTML = this.data.education.map((edu, index) => `
            <div class="education-item">
                <div class="item-header">
                    <span class="item-number">Education ${index + 1}</span>
                    <button type="button" class="remove-item" onclick="app.removeEducation(${index})" title="Remove this education">
                        <i class="fas fa-trash"></i>
                    </button>
                </div>
                
                <div class="form-row">
                    <div class="form-group">
                        <label>Degree</label>
                        <input type="text" placeholder="e.g., Bachelor of Science" value="${edu.degree || ''}" onchange="app.updateEducationField(${index}, 'degree', this.value)">
                    </div>
                    <div class="form-group">
                        <label>School/University</label>
                        <input type="text" placeholder="e.g., University of Example" value="${edu.school || ''}" onchange="app.updateEducationField(${index}, 'school', this.value)">
                    </div>
                </div>
                
                <div class="form-group">
                    <label>Graduation Date</label>
                    <input type="text" placeholder="e.g., May 2020" value="${edu.graduationDate || ''}" onchange="app.updateEducationField(${index}, 'graduationDate', this.value)">
                </div>
                
                <div class="form-group">
                    <label>Additional Details (Optional)</label>
                    <textarea rows="2" placeholder="GPA, honors, relevant coursework..." onchange="app.updateEducationField(${index}, 'details', this.value)">${edu.details || ''}</textarea>
                </div>
            </div>
        `).join('');
    }
    
    updateEducationField(index, field, value) {
        if (this.data.education[index]) {
            this.data.education[index][field] = value;
            this.updatePreview();
            this.saveData();
        }
    }
    
    updateEducationData() {
        // This method is called from updateData() but the actual updates
        // are handled by updateEducationField() for real-time updates
    }
    
    // Preview Management
    updatePreview() {
        const previewContainer = document.getElementById('resume-preview');
        
        if (!previewContainer) return;
        
        try {
            const templateHtml = TemplateUtils.renderTemplate(this.currentTemplate, this.data);
            previewContainer.innerHTML = templateHtml;
            
            // Apply zoom level
            this.applyZoom();
        } catch (error) {
            console.error('Error updating preview:', error);
            previewContainer.innerHTML = `
                <div style="padding: 2rem; text-align: center; color: var(--danger-color);">
                    <i class="fas fa-exclamation-triangle" style="font-size: 2rem; margin-bottom: 1rem;"></i>
                    <p>Error generating preview. Please check your data and try again.</p>
                </div>
            `;
        }
    }
    
    // Zoom Controls
    zoomIn() {
        if (this.zoomLevel < 150) {
            this.zoomLevel += 10;
            this.applyZoom();
        }
    }
    
    zoomOut() {
        if (this.zoomLevel > 50) {
            this.zoomLevel -= 10;
            this.applyZoom();
        }
    }
    
    applyZoom() {
        const preview = document.getElementById('resume-preview');
        const zoomLabel = document.getElementById('zoom-level');
        
        if (preview) {
            preview.style.transform = `scale(${this.zoomLevel / 100})`;
        }
        
        if (zoomLabel) {
            zoomLabel.textContent = `${this.zoomLevel}%`;
        }
    }
    
    // Print and Export
    printResume() {
        // Show loading
        this.showLoading('Preparing your resume for printing...');
        
        setTimeout(() => {
            // Update preview one more time to ensure it's current
            this.updatePreview();
            
            // Trigger print
            window.print();
            
            this.hideLoading();
        }, 500);
    }
    
    // UI Helper Methods
    showLoading(message = 'Loading...') {
        const overlay = document.getElementById('loading-overlay');
        const messageElement = overlay.querySelector('p');
        
        if (messageElement) {
            messageElement.textContent = message;
        }
        
        overlay.style.display = 'flex';
    }
    
    hideLoading() {
        const overlay = document.getElementById('loading-overlay');
        overlay.style.display = 'none';
    }
    
    showToast(type, message) {
        const toastId = type === 'success' ? 'success-toast' : 'error-toast';
        const toast = document.getElementById(toastId);
        const messageElement = toast.querySelector('span');
        
        if (messageElement) {
            messageElement.textContent = message;
        }
        
        toast.style.display = 'flex';
        
        // Auto-hide after 3 seconds
        setTimeout(() => {
            toast.style.display = 'none';
        }, 3000);
    }
    
    showClearConfirmation() {
        document.getElementById('modal-message').textContent = 
            'Are you sure you want to clear all resume data? This action cannot be undone.';
        document.getElementById('confirmation-modal').style.display = 'flex';
        
        // Set up confirmation action
        this.pendingAction = () => {
            this.clearAllData();
            this.hideModal();
        };
    }
    
    hideModal() {
        document.getElementById('confirmation-modal').style.display = 'none';
        this.pendingAction = null;
    }
    
    confirmAction() {
        if (this.pendingAction) {
            this.pendingAction();
        }
    }
    
    clearAllData() {
        // Reset data
        this.data = {
            fullName: '',
            email: '',
            phone: '',
            city: '',
            state: '',
            linkedin: '',
            website: '',
            summary: '',
            experience: [],
            education: [],
            technicalSkills: '',
            softSkills: '',
            languages: '',
            certifications: ''
        };
        
        // Clear localStorage
        localStorage.removeItem('resume-builder-data');
        
        // Reset form
        document.getElementById('resume-form').reset();
        
        // Re-render everything
        this.renderExperience();
        this.renderEducation();
        this.updatePreview();
        
        // Go back to first section
        this.currentSection = 0;
        this.showSection();
        this.updateProgressIndicator();
        
        this.showToast('success', 'All data has been cleared successfully.');
    }
    
    // Keyboard Shortcuts
    handleKeyboardShortcuts(e) {
        // Ctrl/Cmd + S to save
        if ((e.ctrlKey || e.metaKey) && e.key === 's') {
            e.preventDefault();
            this.saveData(true);
        }
        
        // Ctrl/Cmd + P to print
        if ((e.ctrlKey || e.metaKey) && e.key === 'p') {
            e.preventDefault();
            this.printResume();
        }
        
        // Arrow keys for navigation (when not in input field)
        if (!['INPUT', 'TEXTAREA'].includes(e.target.tagName)) {
            if (e.key === 'ArrowRight' || e.key === 'ArrowDown') {
                e.preventDefault();
                this.nextSection();
            } else if (e.key === 'ArrowLeft' || e.key === 'ArrowUp') {
                e.preventDefault();
                this.prevSection();
            }
        }
        
        // Escape to close modals
        if (e.key === 'Escape') {
            this.hideModal();
        }
    }
}

// Initialize the application when DOM is loaded
document.addEventListener('DOMContentLoaded', () => {
    window.app = new ResumeBuilder();
});

// Service Worker registration for offline functionality (optional enhancement)
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/service-worker.js')
            .then(() => console.log('Service Worker registered'))
            .catch(() => console.log('Service Worker registration failed'));
    });
}
