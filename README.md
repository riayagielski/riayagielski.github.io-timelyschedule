[unified_schedule_app.tsx](https://github.com/user-attachments/files/23867354/unified_schedule_app.tsx)
import React, { useState, useEffect } from 'react';
import { Clock, Calendar, CheckCircle, Trash2, Brain, ArrowRight, ArrowLeft, Plus, Save, TrendingUp, RefreshCw, Zap } from 'lucide-react';

const UnifiedScheduleApp = () => {
  // App state management
  const [currentStep, setCurrentStep] = useState('performance');
  const [isLoaded, setIsLoaded] = useState(false);
  
  // Performance data
  const [performanceData, setPerformanceData] = useState({
    typingSpeed: 0,
    readingSpeed: 0,
    typingTime: 0,
    readingTime: 0
  });

  // Quiz data
  const [quizData, setQuizData] = useState({
    classes: [],
    clubs: [],
    flexibleTasks: [],
    wakeTime: '',
    sleepTime: '',
    breakfast: '',
    lunch: '',
    dinner: '',
    productiveTime: 'Morning',
    breakFrequency: 50,
    studyStyle: 'Multiple short sessions'
  });

  // Schedule data
  const [tasks, setTasks] = useState([]);
  const [fixedEvents, setFixedEvents] = useState([]);
  const [draggedTask, setDraggedTask] = useState(null);
  
  // ML data
  const [taskHistory, setTaskHistory] = useState({});
  const [showTimeInput, setShowTimeInput] = useState({});
  const [actualTimeInputs, setActualTimeInputs] = useState({});
  
  // Saved templates
  const [savedClasses, setSavedClasses] = useState([]);
  const [savedClubs, setSavedClubs] = useState([]);
  const [taskTemplates, setTaskTemplates] = useState([]);
  const [showTaskDropdown, setShowTaskDropdown] = useState(false);

  const days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'];
  const hours = Array.from({ length: 17 }, (_, i) => i + 7);

  // Load saved data from localStorage
  useEffect(() => {
    const savedHistory = localStorage.getItem('taskHistory');
    const savedPerformance = localStorage.getItem('performanceData');
    const savedClassTemplates = localStorage.getItem('savedClasses');
    const savedClubTemplates = localStorage.getItem('savedClubs');
    const savedTaskTemplates = localStorage.getItem('taskTemplates');
    const savedHabits = localStorage.getItem('savedHabits');
    
    if (savedHistory) setTaskHistory(JSON.parse(savedHistory));
    if (savedPerformance) setPerformanceData(JSON.parse(savedPerformance));
    if (savedClassTemplates) {
      const templates = JSON.parse(savedClassTemplates);
      setSavedClasses(templates);
      setQuizData(prev => ({ ...prev, classes: templates }));
    }
    if (savedClubTemplates) {
      const templates = JSON.parse(savedClubTemplates);
      setSavedClubs(templates);
      setQuizData(prev => ({ ...prev, clubs: templates }));
    }
    if (savedTaskTemplates) setTaskTemplates(JSON.parse(savedTaskTemplates));
    if (savedHabits) {
      const habits = JSON.parse(savedHabits);
      setQuizData(prev => ({ ...prev, ...habits }));
    }
  }, []);

  // Save data to localStorage
  useEffect(() => {
    if (Object.keys(taskHistory).length > 0) {
      localStorage.setItem('taskHistory', JSON.stringify(taskHistory));
    }
  }, [taskHistory]);

  useEffect(() => {
    if (performanceData.typingSpeed > 0 || performanceData.readingSpeed > 0) {
      localStorage.setItem('performanceData', JSON.stringify(performanceData));
    }
  }, [performanceData]);
  
  useEffect(() => {
    if (quizData.classes.length > 0) {
      localStorage.setItem('savedClasses', JSON.stringify(quizData.classes));
      setSavedClasses(quizData.classes);
    }
  }, [quizData.classes]);
  
  useEffect(() => {
    if (quizData.clubs.length > 0) {
      localStorage.setItem('savedClubs', JSON.stringify(quizData.clubs));
      setSavedClubs(quizData.clubs);
    }
  }, [quizData.clubs]);
  
  useEffect(() => {
    const habits = {
      wakeTime: quizData.wakeTime,
      sleepTime: quizData.sleepTime,
      breakfast: quizData.breakfast,
      lunch: quizData.lunch,
      dinner: quizData.dinner,
      productiveTime: quizData.productiveTime,
      breakFrequency: quizData.breakFrequency,
      studyStyle: quizData.studyStyle
    };
    
    if (quizData.wakeTime || quizData.sleepTime || quizData.breakfast) {
      localStorage.setItem('savedHabits', JSON.stringify(habits));
    }
  }, [quizData.wakeTime, quizData.sleepTime, quizData.breakfast, quizData.lunch, quizData.dinner, quizData.productiveTime, quizData.breakFrequency, quizData.studyStyle]);

  // ========== PERFORMANCE ASSESSMENT ==========

  const [typingText, setTypingText] = useState('');
  const [typingStartTime, setTypingStartTime] = useState(null);
  const [readingStartTime, setReadingStartTime] = useState(null);
  const [readingCompleted, setReadingCompleted] = useState(false);

  const sampleTypingText = "The quick brown fox jumps over the lazy dog while carrying a heavy backpack full of textbooks and notebooks for the upcoming semester.";
  const sampleReadingText = "Effective time management is crucial for academic success. Students who plan their schedules carefully tend to perform better and experience less stress. By breaking large assignments into smaller tasks and scheduling regular study sessions, you can maintain consistent progress throughout the semester. Research shows that students who use structured planning tools are more likely to meet deadlines and achieve their academic goals.";

  const startTypingTest = () => {
    setTypingStartTime(Date.now());
    setTypingText('');
  };

  const completeTypingTest = () => {
    if (typingStartTime && typingText.trim().length > 0) {
      const timeInMinutes = (Date.now() - typingStartTime) / 60000;
      const wordCount = typingText.trim().split(/\s+/).length;
      const wpm = Math.round(wordCount / timeInMinutes);
      
      setPerformanceData(prev => ({
        ...prev,
        typingSpeed: wpm,
        typingTime: timeInMinutes
      }));
      setTypingStartTime(null);
    }
  };

  const startReadingTest = () => {
    setReadingStartTime(Date.now());
    setReadingCompleted(false);
  };

  const completeReadingTest = () => {
    if (readingStartTime) {
      const timeInMinutes = (Date.now() - readingStartTime) / 60000;
      const wordCount = sampleReadingText.split(/\s+/).length;
      const wpm = Math.round(wordCount / timeInMinutes);
      
      setPerformanceData(prev => ({
        ...prev,
        readingSpeed: wpm,
        readingTime: timeInMinutes
      }));
      setReadingStartTime(null);
      setReadingCompleted(true);
    }
  };

  // ========== QUIZ SECTION ==========
  
  const addClass = () => {
    setQuizData({
      ...quizData,
      classes: [...quizData.classes, { name: '', days: ['Monday'], startTime: '09:00', endTime: '10:00' }]
    });
  };

  const updateClass = (index, field, value) => {
    const updated = [...quizData.classes];
    updated[index][field] = value;
    setQuizData({ ...quizData, classes: updated });
  };

  const toggleClassDay = (index, day) => {
    const updated = [...quizData.classes];
    const currentDays = updated[index].days || [updated[index].day];
    
    if (currentDays.includes(day)) {
      updated[index].days = currentDays.filter(d => d !== day);
    } else {
      updated[index].days = [...currentDays, day];
    }
    
    setQuizData({ ...quizData, classes: updated });
  };

  const removeClass = (index) => {
    setQuizData({
      ...quizData,
      classes: quizData.classes.filter((_, i) => i !== index)
    });
  };

  const addClub = () => {
    setQuizData({
      ...quizData,
      clubs: [...quizData.clubs, { name: '', days: ['Monday'], startTime: '15:00', endTime: '16:00' }]
    });
  };

  const updateClub = (index, field, value) => {
    const updated = [...quizData.clubs];
    updated[index][field] = value;
    setQuizData({ ...quizData, clubs: updated });
  };

  const toggleClubDay = (index, day) => {
    const updated = [...quizData.clubs];
    const currentDays = updated[index].days || [updated[index].day];
    
    if (currentDays.includes(day)) {
      updated[index].days = currentDays.filter(d => d !== day);
    } else {
      updated[index].days = [...currentDays, day];
    }
    
    setQuizData({ ...quizData, clubs: updated });
  };

  const removeClub = (index) => {
    setQuizData({
      ...quizData,
      clubs: quizData.clubs.filter((_, i) => i !== index)
    });
  };

  const addFlexibleTask = () => {
    setQuizData({
      ...quizData,
      flexibleTasks: [...quizData.flexibleTasks, { 
        name: '', 
        estimatedTime: 60, 
        dueDate: '', 
        priority: 'medium',
        taskType: 'general',
        requiresOneSitting: false,
        saveAsTemplate: false
      }]
    });
    setShowTaskDropdown(false);
  };
  
  const addTaskFromTemplate = (template) => {
    setQuizData({
      ...quizData,
      flexibleTasks: [...quizData.flexibleTasks, { 
        ...template,
        dueDate: '',
        saveAsTemplate: true
      }]
    });
    setShowTaskDropdown(false);
  };

  const updateFlexibleTask = (index, field, value) => {
    const updated = [...quizData.flexibleTasks];
    updated[index][field] = value;
    
    if (field === 'saveAsTemplate' && value === true && updated[index].name) {
      saveTaskAsTemplate(updated[index]);
    } else if (field === 'saveAsTemplate' && value === false) {
      removeTaskTemplate(updated[index].name);
    }
    
    setQuizData({ ...quizData, flexibleTasks: updated });
  };

  const removeFlexibleTask = (index) => {
    setQuizData({
      ...quizData,
      flexibleTasks: quizData.flexibleTasks.filter((_, i) => i !== index)
    });
  };
  
  const saveTaskAsTemplate = (task) => {
    const template = {
      name: task.name,
      estimatedTime: task.estimatedTime,
      priority: task.priority,
      taskType: task.taskType,
      requiresOneSitting: task.requiresOneSitting
    };
    
    const existingIndex = taskTemplates.findIndex(t => t.name === template.name);
    let updated;
    
    if (existingIndex >= 0) {
      updated = [...taskTemplates];
      updated[existingIndex] = template;
    } else {
      updated = [...taskTemplates, template];
    }
    
    setTaskTemplates(updated);
    localStorage.setItem('taskTemplates', JSON.stringify(updated));
  };
  
  const removeTaskTemplate = (taskName) => {
    const updated = taskTemplates.filter(t => t.name !== taskName);
    setTaskTemplates(updated);
    localStorage.setItem('taskTemplates', JSON.stringify(updated));
  };

  const convertTo24Hour = (time12h) => {
    const [time, modifier] = time12h.trim().split(/\s+/);
    let [hours, minutes] = time.split(':');
    hours = parseInt(hours);
    
    if (modifier && modifier.toUpperCase() === 'PM' && hours !== 12) hours += 12;
    if (modifier && modifier.toUpperCase() === 'AM' && hours === 12) hours = 0;
    
    return `${hours.toString().padStart(2, '0')}:${minutes}`;
  };

  const generateScheduleFromQuiz = () => {
    const fixed = [];
    
    quizData.classes.forEach((cls, idx) => {
      if (cls.name) {
        const classDays = cls.days || [cls.day || 'Monday'];
        classDays.forEach(day => {
          fixed.push({
            id: `class-${idx}-${day}`,
            name: cls.name,
            day: day,
            startTime: cls.startTime,
            endTime: cls.endTime,
            type: 'class'
          });
        });
      }
    });

    quizData.clubs.forEach((club, idx) => {
      if (club.name) {
        const clubDays = club.days || [club.day || 'Monday'];
        clubDays.forEach(day => {
          fixed.push({
            id: `club-${idx}-${day}`,
            name: club.name,
            day: day,
            startTime: club.startTime,
            endTime: club.endTime,
            type: 'club'
          });
        });
      }
    });

    const mealSuggestions = [];
    
    if (quizData.breakfast) {
      const breakfastTime = quizData.breakfast.includes(':') ? 
        (quizData.breakfast.includes('M') ? convertTo24Hour(quizData.breakfast) : quizData.breakfast) :
        '08:00';
      mealSuggestions.push({
        name: 'Breakfast',
        preferredTime: breakfastTime,
        duration: 30,
        type: 'meal'
      });
    }

    if (quizData.lunch) {
      const lunchTime = quizData.lunch.includes(':') ? 
        (quizData.lunch.includes('M') ? convertTo24Hour(quizData.lunch) : quizData.lunch) :
        '12:00';
      mealSuggestions.push({
        name: 'Lunch',
        preferredTime: lunchTime,
        duration: 30,
        type: 'meal'
      });
    }

    if (quizData.dinner) {
      const dinnerTime = quizData.dinner.includes(':') ? 
        (quizData.dinner.includes('M') ? convertTo24Hour(quizData.dinner) : quizData.dinner) :
        '18:00';
      mealSuggestions.push({
        name: 'Dinner',
        preferredTime: dinnerTime,
        duration: 30,
        type: 'meal'
      });
    }
    
    mealSuggestions.forEach((meal) => {
      days.forEach(day => {
        const scheduledMealTime = findMealTime(day, meal.preferredTime, meal.duration, fixed);
        fixed.push({
          id: `${meal.name.toLowerCase()}-${day}`,
          name: meal.name,
          day: day,
          startTime: scheduledMealTime,
          endTime: addMinutes(scheduledMealTime, meal.duration),
          type: 'meal'
        });
      });
    });

    setFixedEvents(fixed);

    const scheduledTasks = autoScheduleTasksWithML(quizData.flexibleTasks, fixed);
    setTasks(scheduledTasks);
    setIsLoaded(true);
    setCurrentStep('schedule');
  };

  const findMealTime = (day, preferredTime, duration, fixedEvents) => {
    const preferredEnd = addMinutes(preferredTime, duration);
    
    const hasConflict = fixedEvents.some(event => {
      if (event.day !== day && event.day !== 'all') return false;
      if (event.type === 'meal') return false;
      return timeOverlaps(preferredTime, preferredEnd, event.startTime, event.endTime);
    });
    
    if (!hasConflict) return preferredTime;
    
    const conflictingEvents = fixedEvents.filter(event => 
      (event.day === day || event.day === 'all') && 
      event.type !== 'meal' &&
      timeOverlaps(preferredTime, preferredEnd, event.startTime, event.endTime)
    );
    
    let latestEndTime = preferredTime;
    conflictingEvents.forEach(event => {
      if (timeToMinutes(event.endTime) > timeToMinutes(latestEndTime)) {
        latestEndTime = event.endTime;
      }
    });
    
    return latestEndTime;
  };

  // ========== ML PREDICTION SYSTEM ==========

  const adjustTimeForPerformance = (baseTime, taskType) => {
    if (taskType === 'typing' && performanceData.typingSpeed > 0) {
      const adjustment = 40 / performanceData.typingSpeed;
      return Math.round(baseTime * adjustment);
    }
    
    if (taskType === 'reading' && performanceData.readingSpeed > 0) {
      const adjustment = 200 / performanceData.readingSpeed;
      return Math.round(baseTime * adjustment);
    }
    
    if (taskType === 'mixed' && performanceData.typingSpeed > 0 && performanceData.readingSpeed > 0) {
      const typingAdj = 40 / performanceData.typingSpeed;
      const readingAdj = 200 / performanceData.readingSpeed;
      const avgAdj = (typingAdj + readingAdj) / 2;
      return Math.round(baseTime * avgAdj);
    }
    
    return baseTime;
  };

  const predictTaskDuration = (taskName, estimatedTime, taskType = 'general') => {
    const normalizedName = taskName.toLowerCase().trim();
    
    let adjustedTime = adjustTimeForPerformance(estimatedTime, taskType);
    
    if (taskHistory[normalizedName] && taskHistory[normalizedName].length > 0) {
      const history = taskHistory[normalizedName];
      
      let weightedSum = 0;
      let weightSum = 0;
      
      history.forEach((entry, index) => {
        const weight = index + 1;
        weightedSum += entry.actualTime * weight;
        weightSum += weight;
      });
      
      const predictedTime = Math.round(weightedSum / weightSum);
      
      const variance = calculateVariance(history.map(h => h.actualTime));
      const adjustment = variance > 10 ? 1.1 : 1.0;
      
      return Math.round(predictedTime * adjustment);
    }
    
    const similarTasks = Object.keys(taskHistory).filter(key => 
      key.includes(normalizedName) || normalizedName.includes(key)
    );
    
    if (similarTasks.length > 0) {
      const allSimilarData = similarTasks.flatMap(key => taskHistory[key]);
      const avgTime = allSimilarData.reduce((sum, entry) => sum + entry.actualTime, 0) / allSimilarData.length;
      return Math.round(avgTime * 1.1);
    }
    
    return adjustedTime;
  };

  const calculateVariance = (values) => {
    if (values.length === 0) return 0;
    const mean = values.reduce((sum, val) => sum + val, 0) / values.length;
    const squaredDiffs = values.map(val => Math.pow(val - mean, 2));
    return Math.sqrt(squaredDiffs.reduce((sum, val) => sum + val, 0) / values.length);
  };

  const recordActualTime = (taskId, taskName, estimatedTime, actualTime) => {
    const normalizedName = taskName.toLowerCase().trim();
    
    const newEntry = {
      date: new Date().toISOString(),
      estimatedTime,
      actualTime,
      accuracy: Math.abs(estimatedTime - actualTime) / estimatedTime
    };

    setTaskHistory(prev => {
      const updated = { ...prev };
      if (!updated[normalizedName]) {
        updated[normalizedName] = [];
      }
      updated[normalizedName] = [...updated[normalizedName], newEntry].slice(-10);
      return updated;
    });

    setShowTimeInput(prev => ({ ...prev, [taskId]: false }));
    setActualTimeInputs(prev => ({ ...prev, [taskId]: '' }));
  };

  const refreshSchedule = () => {
    const scheduledTasks = autoScheduleTasksWithML(
      tasks.map(t => ({
        name: t.name,
        estimatedTime: t.originalEstimate,
        dueDate: t.dueDate,
        priority: t.priority,
        taskType: t.taskType,
        requiresOneSitting: t.requiresOneSitting
      })),
      fixedEvents
    );
    setTasks(scheduledTasks);
    alert('Schedule refreshed with updated AI predictions!');
  };

  // ========== SCHEDULING LOGIC ==========

  const addMinutes = (time, mins) => {
    const [h, m] = time.split(':').map(Number);
    const totalMins = h * 60 + m + mins;
    const newH = Math.floor(totalMins / 60) % 24;
    const newM = totalMins % 60;
    return `${newH.toString().padStart(2, '0')}:${newM.toString().padStart(2, '0')}`;
  };

  const autoScheduleTasksWithML = (taskList, fixedList) => {
    const breakFreq = quizData.breakFrequency || 50;
    const productiveHours = getProductiveHours(quizData.productiveTime);
    
    const sortedTasks = [...taskList].sort((a, b) => {
      const priorityOrder = { high: 0, medium: 1, low: 2 };
      if (priorityOrder[a.priority] !== priorityOrder[b.priority]) {
        return priorityOrder[a.priority] - priorityOrder[b.priority];
      }
      if (a.dueDate && b.dueDate) return new Date(a.dueDate) - new Date(b.dueDate);
      return a.dueDate ? -1 : 1;
    });

    const dailyWorkload = {};
    days.forEach(day => dailyWorkload[day] = 0);

    return sortedTasks.map((task, taskIdx) => {
      const predictedDuration = predictTaskDuration(task.name, task.estimatedTime, task.taskType || 'general');
      
      const slots = [];
      let remainingTime = predictedDuration;

      if (task.requiresOneSitting) {
        const daysByWorkload = [...days].sort((a, b) => dailyWorkload[a] - dailyWorkload[b]);
        
        for (const day of daysByWorkload) {
          if (slots.length === 0) {
            const availableSlot = findAvailableSlot(day, productiveHours, predictedDuration, fixedList, []);
            
            if (availableSlot) {
              slots.push({
                day,
                startTime: availableSlot,
                duration: predictedDuration
              });
              dailyWorkload[day] += predictedDuration;
              remainingTime = 0;
            }
          }
        }
      } else {
        while (remainingTime > 0) {
          const sessionTime = Math.min(remainingTime, breakFreq);
          
          const daysByWorkload = [...days].sort((a, b) => dailyWorkload[a] - dailyWorkload[b]);
          
          let slotFound = false;
          for (const day of daysByWorkload) {
            const availableSlot = findAvailableSlot(day, productiveHours, sessionTime, fixedList, slots);
            
            if (availableSlot) {
              slots.push({
                day,
                startTime: availableSlot,
                duration: sessionTime
              });
              dailyWorkload[day] += sessionTime;
              remainingTime -= sessionTime;
              slotFound = true;
              break;
            }
          }
          
          if (!slotFound) break;
        }
      }

      return {
        id: `task-${taskIdx}`,
        name: task.name,
        originalEstimate: task.estimatedTime,
        predictedDuration: predictedDuration,
        dueDate: task.dueDate,
        priority: task.priority,
        taskType: task.taskType || 'general',
        requiresOneSitting: task.requiresOneSitting || false,
        type: 'flexible',
        scheduledSlots: slots,
        actualTime: null
      };
    });
  };

  const getProductiveHours = (productiveTime) => {
    if (!productiveTime) return { start: 8, end: 12 };
    
    if (productiveTime.toLowerCase().includes('morning')) return { start: 8, end: 12 };
    if (productiveTime.toLowerCase().includes('afternoon')) return { start: 13, end: 17 };
    if (productiveTime.toLowerCase().includes('evening')) return { start: 18, end: 22 };
    
    return { start: 8, end: 12 };
  };

  const findAvailableSlot = (day, productiveHours, duration, fixedList, existingSlots) => {
    const startHour = productiveHours.start;
    const endHour = productiveHours.end;
    
    for (let hour = startHour; hour < endHour; hour++) {
      const slotStart = `${hour.toString().padStart(2, '0')}:00`;
      const slotEnd = addMinutes(slotStart, duration);
      
      const hasConflict = fixedList.some(event => {
        if (event.day !== day && event.day !== 'all') return false;
        return timeOverlaps(slotStart, slotEnd, event.startTime, event.endTime);
      });
      
      const hasSlotConflict = existingSlots.some(slot => {
        if (slot.day !== day) return false;
        const existingEnd = addMinutes(slot.startTime, slot.duration);
        return timeOverlaps(slotStart, slotEnd, slot.startTime, existingEnd);
      });

      if (!hasConflict && !hasSlotConflict) return slotStart;
    }
    
    return null;
  };

  const timeOverlaps = (start1, end1, start2, end2) => {
    const s1 = timeToMinutes(start1);
    const e1 = timeToMinutes(end1);
    const s2 = timeToMinutes(start2);
    const e2 = timeToMinutes(end2);
    return s1 < e2 && e1 > s2;
  };

  const timeToMinutes = (time) => {
    const [hours, minutes] = time.split(':').map(Number);
    return hours * 60 + minutes;
  };

  const minutesToTime = (minutes) => {
    const hours = Math.floor(minutes / 60);
    const mins = minutes % 60;
    return `${hours.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')}`;
  };

  const handleDragStart = (task) => {
    setDraggedTask(task);
  };

  const handleDrop = (day, hour) => {
    if (!draggedTask) return;

    const startMinutes = hour * 60;
    const duration = Math.min(draggedTask.predictedDuration, quizData.breakFrequency || 50);
    const newSlot = {
      day,
      startTime: minutesToTime(startMinutes),
      duration
    };

    setTasks(tasks.map(t => 
      t.id === draggedTask.id 
        ? { ...t, scheduledSlots: [...t.scheduledSlots, newSlot] }
        : t
    ));
    setDraggedTask(null);
  };

  const removeSlot = (taskId, slotIndex) => {
    setTasks(tasks.map(t => 
      t.id === taskId 
        ? { ...t, scheduledSlots: t.scheduledSlots.filter((_, i) => i !== slotIndex) }
        : t
    ));
  };

  const getAllEventsForDay = (day) => {
    const events = [];

    fixedEvents.forEach(event => {
      if (event.day === day || event.day === 'all') {
        events.push({
          ...event,
          start: timeToMinutes(event.startTime),
          end: timeToMinutes(event.endTime),
          isFixed: true,
          uniqueId: event.id
        });
      }
    });

    tasks.forEach(task => {
      task.scheduledSlots.forEach((slot, index) => {
        if (slot.day === day) {
          const start = timeToMinutes(slot.startTime);
          events.push({
            id: `${task.id}-${index}`,
            name: task.name,
            start,
            end: start + slot.duration,
            type: task.type,
            priority: task.priority,
            isFixed: false,
            taskId: task.id,
            slotIndex: index,
            uniqueId: `${task.id}-${index}`
          });
        }
      });
    });

    const sortedEvents = events.sort((a, b) => a.start - b.start);
    const groupedEvents = [];
    
    sortedEvents.forEach(event => {
      let addedToGroup = false;
      
      for (let group of groupedEvents) {
        const hasOverlap = group.some(e => 
          timeOverlaps(
            minutesToTime(event.start), 
            minutesToTime(event.end),
            minutesToTime(e.start),
            minutesToTime(e.end)
          )
        );
        
        if (hasOverlap) {
          group.push(event);
          addedToGroup = true;
          break;
        }
      }
      
      if (!addedToGroup) {
        groupedEvents.push([event]);
      }
    });
    
    const eventsWithColumns = [];
    groupedEvents.forEach(group => {
      const columnCount = group.length;
      group.forEach((event, index) => {
        eventsWithColumns.push({
          ...event,
          columnIndex: index,
          columnCount: columnCount
        });
      });
    });

    return eventsWithColumns;
  };

  const getColorForEvent = (event) => {
    if (event.type === 'class') return 'bg-blue-500';
    if (event.type === 'club') return 'bg-purple-500';
    if (event.type === 'meal') return 'bg-orange-400';
    if (event.priority === 'high') return 'bg-red-500';
    if (event.priority === 'medium') return 'bg-yellow-500';
    return 'bg-green-500';
  };

  const formatTime = (minutes) => {
    const hours = Math.floor(minutes / 60);
    const mins = minutes % 60;
    const period = hours >= 12 ? 'PM' : 'AM';
    const displayHours = hours > 12 ? hours - 12 : hours === 0 ? 12 : hours;
    return `${displayHours}:${mins.toString().padStart(2, '0')} ${period}`;
  };

  const getTotalScheduled = (task) => {
    return task.scheduledSlots.reduce((sum, slot) => sum + slot.duration, 0);
  };

  const resetApp = () => {
    setCurrentStep('performance');
    setIsLoaded(false);
    setTasks([]);
    setFixedEvents([]);
  };

  // ========== PERFORMANCE TEST UI ==========

  if (currentStep === 'performance') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 p-6">
        <div className="max-w-4xl mx-auto">
          <div className="bg-white rounded-2xl shadow-2xl p-8">
            <div className="text-center mb-8">
              <div className="flex justify-center mb-4">
                <div className="bg-indigo-100 p-6 rounded-full">
                  <Zap className="text-indigo-600" size={48} />
                </div>
              </div>
              <h1 className="text-4xl font-bold text-gray-800 mb-2">Performance Assessment</h1>
              <p className="text-gray-600">Let's measure your typing and reading speed to optimize predictions</p>
            </div>

            <div className="mb-8">
              <h2 className="text-2xl font-bold text-gray-800 mb-4">Typing Speed Test</h2>
              <div className="bg-gray-50 p-6 rounded-xl mb-4">
                <p className="text-gray-700 mb-4 font-mono text-sm bg-white p-4 rounded border">
                  {sampleTypingText}
                </p>
                <p className="text-sm text-gray-600 mb-3">Type the text above as quickly and accurately as you can:</p>
                
                {!typingStartTime ? (
                  <button
                    onClick={startTypingTest}
                    className="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-lg font-semibold"
                  >
                    Start Typing Test
                  </button>
                ) : (
                  <div>
                    <textarea
                      value={typingText}
                      onChange={(e) => setTypingText(e.target.value)}
                      className="w-full h-32 px-4 py-3 border-2 border-indigo-300 rounded-lg mb-3 font-mono text-sm"
                      placeholder="Start typing here..."
                      autoFocus
                    />
                    <button
                      onClick={completeTypingTest}
                      disabled={typingText.trim().length === 0}
                      className="bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-lg font-semibold disabled:bg-gray-400"
                    >
                      Complete Test
                    </button>
                  </div>
                )}
                
                {performanceData.typingSpeed > 0 && (
                  <div className="mt-4 p-4 bg-green-50 border border-green-200 rounded-lg">
                    <p className="text-green-800 font-semibold">
                      ✓ Typing Speed: {performanceData.typingSpeed} WPM
                    </p>
                  </div>
                )}
              </div>
            </div>

            <div className="mb-8">
              <h2 className="text-2xl font-bold text-gray-800 mb-4">Reading Speed Test</h2>
              <div className="bg-gray-50 p-6 rounded-xl mb-4">
                <p className="text-sm text-gray-600 mb-3">Click start, read the passage below carefully, then click finish:</p>
                
                {!readingStartTime && !readingCompleted && (
                  <button
                    onClick={startReadingTest}
                    className="bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-lg font-semibold mb-4"
                  >
                    Start Reading Test
                  </button>
                )}
                
                <div className="bg-white p-6 rounded-lg border mb-4">
                  <p className="text-gray-700 leading-relaxed">
                    {sampleReadingText}
                  </p>
                </div>
                
                {readingStartTime && !readingCompleted && (
                  <button
                    onClick={completeReadingTest}
                    className="bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-lg font-semibold"
                  >
                    Finish Reading Test
                  </button>
                )}
                
                {performanceData.readingSpeed > 0 && (
                  <div className="mt-4 p-4 bg-green-50 border border-green-200 rounded-lg">
                    <p className="text-green-800 font-semibold">
                      ✓ Reading Speed: {performanceData.readingSpeed} WPM
                    </p>
                  </div>
                )}
              </div>
            </div>

            <div className="flex gap-4">
              <button
                onClick={() => setCurrentStep('quiz')}
                disabled={performanceData.typingSpeed === 0 || performanceData.readingSpeed === 0}
                className="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white py-4 rounded-xl font-bold text-lg flex items-center justify-center gap-2 transition-all disabled:bg-gray-400"
              >
                Continue to Schedule Setup <ArrowRight />
              </button>
              <button
                onClick={() => setCurrentStep('quiz')}
                className="bg-gray-500 hover:bg-gray-600 text-white px-6 py-4 rounded-xl font-semibold"
              >
                Skip Tests
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }

  // ========== QUIZ UI ==========

  if (currentStep === 'quiz') {
    return (
      <div className="min-h-screen bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 p-6">
        <div className="max-w-4xl mx-auto">
          <div className="bg-white rounded-2xl shadow-2xl p-8">
            <div className="text-center mb-8">
              <h1 className="text-4xl font-bold text-gray-800 mb-2">Student Schedule Optimizer</h1>
              <p className="text-gray-600">AI-powered time management with machine learning</p>
              {performanceData.typingSpeed > 0 && (
                <div className="mt-3 flex justify-center gap-4 text-sm">
                  <span className="bg-indigo-100 text-indigo-700 px-3 py-1 rounded-full">
                    Typing: {performanceData.typingSpeed} WPM
                  </span>
                  <span className="bg-purple-100 text-purple-700 px-3 py-1 rounded-full">
                    Reading: {performanceData.readingSpeed} WPM
                  </span>
                </div>
              )}
            </div>

            <div className="mb-8">
              <h2 className="text-2xl font-bold text-gray-800 mb-4 flex items-center gap-2">
                <Calendar className="text-indigo-600" />
                Fixed Schedule Items
              </h2>
              
              <div className="mb-6">
                <div className="flex justify-between items-center mb-3">
                  <h3 className="text-lg font-semibold text-gray-700">Classes</h3>
                  <button onClick={addClass} className="bg-indigo-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-indigo-700">
                    <Plus size={16} /> Add Class
                  </button>
                </div>
                {quizData.classes.map((cls, idx) => (
                  <div key={idx} className="bg-gray-50 p-4 rounded-lg mb-3">
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-3 mb-3">
                      <input
                        type="text"
                        placeholder="Class name"
                        value={cls.name}
                        onChange={(e) => updateClass(idx, 'name', e.target.value)}
                        className="px-3 py-2 border rounded-lg"
                      />
                      <input
                        type="time"
                        value={cls.startTime}
                        onChange={(e) => updateClass(idx, 'startTime', e.target.value)}
                        className="px-3 py-2 border rounded-lg"
                      />
                      <input
                        type="time"
                        value={cls.endTime}
                        onChange={(e) => updateClass(idx, 'endTime', e.target.value)}
                        className="px-3 py-2 border rounded-lg"
                      />
                    </div>
                    <div className="mb-2">
                      <label className="block text-sm font-medium text-gray-700 mb-2">Select Days:</label>
                      <div className="flex flex-wrap gap-2">
                        {days.map(day => {
                          const classDays = cls.days || [cls.day || 'Monday'];
                          const isSelected = classDays.includes(day);
                          return (
                            <button
                              key={day}
                              onClick={() => toggleClassDay(idx, day)}
                              className={`px-3 py-1.5 rounded-lg text-sm font-medium transition-all ${
                                isSelected 
                                  ? 'bg-indigo-600 text-white' 
                                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                              }`}
                            >
                              {day.substring(0, 3)}
                            </button>
                          );
                        })}
                      </div>
                    </div>
                    <button onClick={() => removeClass(idx)} className="text-red-500 text-sm mt-2 hover:text-red-700">
                      Remove
                    </button>
                  </div>
                ))}
              </div>

              <div className="mb-6">
                <div className="flex justify-between items-center mb-3">
                  <h3 className="text-lg font-semibold text-gray-700">Clubs/Activities</h3>
                  <button onClick={addClub} className="bg-purple-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-purple-700">
                    <Plus size={16} /> Add Club
                  </button>
                </div>
                {quizData.clubs.map((club, idx) => (
                  <div key={idx} className="bg-gray-50 p-4 rounded-lg mb-3">
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-3 mb-3">
                      <input
                        type="text"
                        placeholder="Club name"
                        value={club.name}
                        onChange={(e) => updateClub(idx, 'name', e.target.value)}
                        className="px-3 py-2 border rounded-lg"
                      />
                      <input
                        type="time"
                        value={club.startTime}
                        onChange={(e) => updateClub(idx, 'startTime', e.target.value)}
                        className="px-3 py-2 border rounded-lg"
                      />
                      <input
                        type="time"
                        value={club.endTime}
                        onChange={(e) => updateClub(idx, 'endTime', e.target.value)}
                        className="px-3 py-2 border rounded-lg"
                      />
                    </div>
                    <div className="mb-2">
                      <label className="block text-sm font-medium text-gray-700 mb-2">Select Days:</label>
                      <div className="flex flex-wrap gap-2">
                        {days.map(day => {
                          const clubDays = club.days || [club.day || 'Monday'];
                          const isSelected = clubDays.includes(day);
                          return (
                            <button
                              key={day}
                              onClick={() => toggleClubDay(idx, day)}
                              className={`px-3 py-1.5 rounded-lg text-sm font-medium transition-all ${
                                isSelected 
                                  ? 'bg-purple-600 text-white' 
                                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                              }`}
                            >
                              {day.substring(0, 3)}
                            </button>
                          );
                        })}
                      </div>
                    </div>
                    <button onClick={() => removeClub(idx)} className="text-red-500 text-sm mt-2 hover:text-red-700">
                      Remove
                    </button>
                  </div>
                ))}
              </div>
            </div>

            <div className="mb-8">
              <h2 className="text-2xl font-bold text-gray-800 mb-4 flex items-center gap-2">
                <CheckCircle className="text-indigo-600" />
                Flexible Tasks & Assignments
              </h2>
              <div className="flex justify-between items-center mb-3">
                <p className="text-sm text-gray-600">Tasks that can be scheduled flexibly</p>
                <div className="relative">
                  {taskTemplates.length > 0 ? (
                    <>
                      <button 
                        onClick={() => setShowTaskDropdown(!showTaskDropdown)}
                        className="bg-green-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-green-700"
                      >
                        <Plus size={16} /> Add Task
                      </button>
                      {showTaskDropdown && (
                        <div className="absolute right-0 mt-2 w-64 bg-white rounded-lg shadow-xl border-2 border-green-500 z-10">
                          <div className="p-2">
                            <button
                              onClick={addFlexibleTask}
                              className="w-full text-left px-3 py-2 hover:bg-gray-100 rounded text-sm font-medium"
                            >
                              ➕ New Custom Task
                            </button>
                            <div className="border-t my-2"></div>
                            <div className="text-xs font-semibold text-gray-500 px-3 py-1">SAVED TEMPLATES</div>
                            {taskTemplates.map((template, idx) => (
                              <button
                                key={idx}
                                onClick={() => addTaskFromTemplate(template)}
                                className="w-full text-left px-3 py-2 hover:bg-green-50 rounded text-sm"
                              >
                                <div className="font-medium">{template.name}</div>
                                <div className="text-xs text-gray-500">
                                  {template.estimatedTime}m • {template.priority} • {template.requiresOneSitting ? 'One sitting' : 'Flexible'}
                                </div>
                              </button>
                            ))}
                          </div>
                        </div>
                      )}
                    </>
                  ) : (
                    <button 
                      onClick={addFlexibleTask}
                      className="bg-green-600 text-white px-4 py-2 rounded-lg flex items-center gap-2 hover:bg-green-700"
                    >
                      <Plus size={16} /> Add Task
                    </button>
                  )}
                </div>
              </div>
              {quizData.flexibleTasks.map((task, idx) => (
                <div key={idx} className="bg-gray-50 p-4 rounded-lg mb-3">
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-3 mb-2">
                    <input
                      type="text"
                      placeholder="Task name (e.g., Bio Lab Report)"
                      value={task.name}
                      onChange={(e) => updateFlexibleTask(idx, 'name', e.target.value)}
                      className="px-3 py-2 border rounded-lg"
                    />
                    <input
                      type="number"
                      placeholder="Estimated time (minutes)"
                      value={task.estimatedTime}
                      onChange={(e) => updateFlexibleTask(idx, 'estimatedTime', parseInt(e.target.value))}
                      className="px-3 py-2 border rounded-lg"
                    />
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-3 mb-3">
                    <input
                      type="date"
                      value={task.dueDate}
                      onChange={(e) => updateFlexibleTask(idx, 'dueDate', e.target.value)}
                      className="px-3 py-2 border rounded-lg"
                    />
                    <select
                      value={task.priority}
                      onChange={(e) => updateFlexibleTask(idx, 'priority', e.target.value)}
                      className="px-3 py-2 border rounded-lg"
                    >
                      <option value="high">High Priority</option>
                      <option value="medium">Medium Priority</option>
                      <option value="low">Low Priority</option>
                    </select>
                    <select
                      value={task.taskType || 'general'}
                      onChange={(e) => updateFlexibleTask(idx, 'taskType', e.target.value)}
                      className="px-3 py-2 border rounded-lg"
                    >
                      <option value="general">General Task</option>
                      <option value="typing">Typing-Heavy (Essay, Code)</option>
                      <option value="reading">Reading-Heavy (Study, Research)</option>
                      <option value="mixed">Mixed (Reading + Writing)</option>
                    </select>
                  </div>
                  
                  <div className="flex items-center gap-4 mb-2">
                    <label className="flex items-center gap-2 cursor-pointer">
                      <input
                        type="checkbox"
                        checked={task.requiresOneSitting || false}
                        onChange={(e) => updateFlexibleTask(idx, 'requiresOneSitting', e.target.checked)}
                        className="w-4 h-4 text-green-600 rounded"
                      />
                      <span className="text-sm font-medium text-gray-700">
                        Must be completed in one sitting
                      </span>
                    </label>
                    
                    <label className="flex items-center gap-2 cursor-pointer ml-auto">
                      <input
                        type="checkbox"
                        checked={task.saveAsTemplate || false}
                        onChange={(e) => updateFlexibleTask(idx, 'saveAsTemplate', e.target.checked)}
                        disabled={!task.name}
                        className="w-4 h-4 text-blue-600 rounded disabled:opacity-50"
                      />
                      <span className="text-sm font-medium text-gray-700 flex items-center gap-1">
                        💾 Save as template
                      </span>
                    </label>
                  </div>
                  
                  <button 
                    onClick={() => removeFlexibleTask(idx)} 
                    className="text-red-500 text-sm hover:text-red-700"
                  >
                    Remove
                  </button>
                </div>
              ))}
            </div>

            <div className="mb-8">
              <h2 className="text-2xl font-bold text-gray-800 mb-4 flex items-center gap-2">
                <Clock className="text-indigo-600" />
                Daily Habits & Preferences
              </h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Wake Up Time</label>
                  <input
                    type="time"
                    value={quizData.wakeTime}
                    onChange={(e) => setQuizData({ ...quizData, wakeTime: e.target.value })}
                    className="w-full px-3 py-2 border rounded-lg"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Sleep Time</label>
                  <input
                    type="time"
                    value={quizData.sleepTime}
                    onChange={(e) => setQuizData({ ...quizData, sleepTime: e.target.value })}
                    className="w-full px-3 py-2 border rounded-lg"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Breakfast Time</label>
                  <input
                    type="time"
                    value={quizData.breakfast}
                    onChange={(e) => setQuizData({ ...quizData, breakfast: e.target.value })}
                    className="w-full px-3 py-2 border rounded-lg"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Lunch Time</label>
                  <input
                    type="time"
                    value={quizData.lunch}
                    onChange={(e) => setQuizData({ ...quizData, lunch: e.target.value })}
                    className="w-full px-3 py-2 border rounded-lg"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Dinner Time</label>
                  <input
                    type="time"
                    value={quizData.dinner}
                    onChange={(e) => setQuizData({ ...quizData, dinner: e.target.value })}
                    className="w-full px-3 py-2 border rounded-lg"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Most Productive Time</label>
                  <select
                    value={quizData.productiveTime}
                    onChange={(e) => setQuizData({ ...quizData, productiveTime: e.target.value })}
                    className="w-full px-3 py-2 border rounded-lg"
                  >
                    <option value="Morning">Morning (8 AM - 12 PM)</option>
                    <option value="Afternoon">Afternoon (1 PM - 5 PM)</option>
                    <option value="Evening">Evening (6 PM - 10 PM)</option>
                  </select>
                </div>
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">Work Session Length (minutes)</label>
                  <input
                    type="number"
                    value={quizData.breakFrequency}
                    onChange={(e) => setQuizData({ ...quizData, breakFrequency: parseInt(e.target.value) })}
                    className="w-full px-3 py-2 border rounded-lg"
                  />
                </div>
              </div>
            </div>

            <div className="flex gap-4">
              <button
                onClick={() => setCurrentStep('performance')}
                className="bg-gray-500 hover:bg-gray-600 text-white px-6 py-4 rounded-xl font-semibold flex items-center gap-2"
              >
                <ArrowLeft size={20} /> Back
              </button>
              <button
                onClick={generateScheduleFromQuiz}
                className="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white py-4 rounded-xl font-bold text-lg flex items-center justify-center gap-2 transition-all"
              >
                Generate AI-Powered Schedule <ArrowRight />
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }

  // ========== SCHEDULE UI ==========

  return (
    <div className="min-h-screen bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 p-6">
      <div className="max-w-7xl mx-auto">
        <div className="bg-white rounded-2xl shadow-lg p-6 mb-6">
          <div className="flex items-center justify-between">
            <div>
              <h1 className="text-3xl font-bold text-gray-800 flex items-center gap-3">
                <Brain className="text-indigo-600" size={36} />
                AI-Enhanced Weekly Schedule
              </h1>
              <p className="text-gray-600 mt-2">
                {quizData.productiveTime && `Optimized for ${quizData.productiveTime.toLowerCase()}`}
                {quizData.breakFrequency && ` • ${quizData.breakFrequency}-min work sessions`}
                {Object.keys(taskHistory).length > 0 && ` • Learning from ${Object.keys(taskHistory).length} tracked tasks`}
              </p>
            </div>
            <div className="flex gap-3">
              <button
                onClick={refreshSchedule}
                className="bg-green-600 hover:bg-green-700 text-white px-6 py-3 rounded-xl font-semibold flex items-center gap-2 transition-all"
              >
                <RefreshCw size={20} />
                Refresh Schedule
              </button>
              <button
                onClick={resetApp}
                className="bg-gray-600 hover:bg-gray-700 text-white px-6 py-3 rounded-xl font-semibold flex items-center gap-2 transition-all"
              >
                <ArrowLeft size={20} />
                Start Over
              </button>
            </div>
          </div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-4 gap-6">
          <div className="bg-white rounded-2xl shadow-lg p-6">
            <h2 className="text-xl font-bold text-gray-800 mb-4 flex items-center gap-2">
              <CheckCircle className="text-indigo-600" />
              Tasks
            </h2>
            <div className="space-y-3 max-h-[600px] overflow-y-auto">
              {tasks.map(task => {
                const scheduled = getTotalScheduled(task);
                const progress = (scheduled / task.predictedDuration) * 100;
                const hasHistory = taskHistory[task.name.toLowerCase().trim()];
                
                return (
                  <div
                    key={task.id}
                    draggable
                    onDragStart={() => handleDragStart(task)}
                    className="bg-gradient-to-r from-gray-50 to-gray-100 p-4 rounded-xl cursor-move hover:shadow-md transition-all border-2 border-transparent hover:border-indigo-300"
                  >
                    <div className="flex justify-between items-start mb-2">
                      <div className="flex-1">
                        <h3 className="font-semibold text-gray-800 text-sm">{task.name}</h3>
                        {task.requiresOneSitting && (
                          <span className="text-xs bg-orange-100 text-orange-700 px-2 py-0.5 rounded-full inline-block mt-1">
                            One sitting required
                          </span>
                        )}
                      </div>
                      <span className={`px-2 py-1 rounded-full text-xs font-semibold ${
                        task.priority === 'high' ? 'bg-red-100 text-red-700' :
                        task.priority === 'medium' ? 'bg-yellow-100 text-yellow-700' :
                        'bg-green-100 text-green-700'
                      }`}>
                        {task.priority}
                      </span>
                    </div>
                    
                    <div className="text-xs text-gray-600 space-y-1">
                      <div className="flex justify-between">
                        <span>Est: {task.originalEstimate}m</span>
                        {task.dueDate && <span>Due: {new Date(task.dueDate).toLocaleDateString('en-US', { month: 'short', day: 'numeric' })}</span>}
                      </div>
                      {hasHistory && (
                        <div className="flex items-center gap-1 text-indigo-600">
                          <TrendingUp size={12} />
                          <span>AI: {task.predictedDuration}m</span>
                        </div>
                      )}
                      
                      <div className="mt-2">
                        <div className="flex justify-between text-xs mb-1">
                          <span>{scheduled}/{task.predictedDuration}m</span>
                          <span>{Math.round(progress)}%</span>
                        </div>
                        <div className="w-full bg-gray-200 rounded-full h-1.5">
                          <div
                            className="bg-indigo-600 h-1.5 rounded-full transition-all"
                            style={{ width: `${Math.min(progress, 100)}%` }}
                          />
                        </div>
                      </div>
                    </div>

                    {!showTimeInput[task.id] && (
                      <button
                        onClick={() => setShowTimeInput(prev => ({ ...prev, [task.id]: true }))}
                        className="mt-2 w-full bg-green-500 hover:bg-green-600 text-white text-xs py-2 rounded-lg flex items-center justify-center gap-1"
                      >
                        <Clock size={12} />
                        Log Time
                      </button>
                    )}

                    {showTimeInput[task.id] && (
                      <div className="mt-2 p-2 bg-green-50 rounded-lg border border-green-200">
                        <label className="block text-xs font-medium text-gray-700 mb-1">
                          Actual time (min):
                        </label>
                        <div className="flex gap-1">
                          <input
                            type="number"
                            value={actualTimeInputs[task.id] || ''}
                            onChange={(e) => setActualTimeInputs(prev => ({ ...prev, [task.id]: e.target.value }))}
                            className="flex-1 px-2 py-1 border border-green-300 rounded text-xs"
                            placeholder="65"
                          />
                          <button
                            onClick={() => {
                              const actualTime = parseInt(actualTimeInputs[task.id]);
                              if (actualTime && actualTime > 0) {
                                recordActualTime(task.id, task.name, task.predictedDuration, actualTime);
                                alert(`Time logged! Click "Refresh Schedule" to update predictions.`);
                              }
                            }}
                            className="bg-green-600 hover:bg-green-700 text-white px-3 py-1 rounded text-xs flex items-center gap-1"
                          >
                            <Save size={12} />
                          </button>
                        </div>
                        <button
                          onClick={() => setShowTimeInput(prev => ({ ...prev, [task.id]: false }))}
                          className="text-xs text-gray-500 hover:text-gray-700 mt-1"
                        >
                          Cancel
                        </button>
                      </div>
                    )}
                  </div>
                );
              })}
            </div>
          </div>

          <div className="lg:col-span-3 bg-white rounded-2xl shadow-lg p-6">
            <div className="mb-4">
              <h2 className="text-xl font-bold text-gray-800 mb-3">Weekly Overview</h2>
              
              <div className="flex flex-wrap gap-2 mb-4 text-xs">
                <div className="flex items-center gap-1">
                  <div className="w-3 h-3 bg-blue-500 rounded"></div>
                  <span>Class</span>
                </div>
                <div className="flex items-center gap-1">
                  <div className="w-3 h-3 bg-purple-500 rounded"></div>
                  <span>Club</span>
                </div>
                <div className="flex items-center gap-1">
                  <div className="w-3 h-3 bg-orange-400 rounded"></div>
                  <span>Meal</span>
                </div>
                <div className="flex items-center gap-1">
                  <div className="w-3 h-3 bg-red-500 rounded"></div>
                  <span>High</span>
                </div>
                <div className="flex items-center gap-1">
                  <div className="w-3 h-3 bg-yellow-500 rounded"></div>
                  <span>Medium</span>
                </div>
                <div className="flex items-center gap-1">
                  <div className="w-3 h-3 bg-green-500 rounded"></div>
                  <span>Low</span>
                </div>
              </div>
            </div>

            <div className="border-2 border-gray-200 rounded-xl overflow-x-auto">
              <div className="min-w-[900px]">
                <div className="grid grid-cols-8 bg-gray-50 border-b-2 border-gray-200">
                  <div className="p-2 font-semibold text-sm text-gray-600 border-r border-gray-200">Time</div>
                  {days.map(day => (
                    <div key={day} className="p-2 font-semibold text-sm text-gray-700 text-center border-r border-gray-200 last:border-r-0">
                      {day.substring(0, 3)}
                    </div>
                  ))}
                </div>

                <div className="max-h-[600px] overflow-y-auto">
                  {hours.map(hour => {
                    return (
                      <div key={hour} className="grid grid-cols-8 border-b border-gray-200 hover:bg-blue-50 transition-colors relative" style={{ height: '60px' }}>
                        <div className="p-2 text-xs font-semibold text-gray-600 bg-gray-50 border-r border-gray-200 flex items-start">
                          {hour > 12 ? hour - 12 : hour === 0 ? 12 : hour}:00 {hour >= 12 ? 'PM' : 'AM'}
                        </div>
                        
                        {days.map(day => {
                          const hourStart = hour * 60;
                          const hourEnd = (hour + 1) * 60;
                          const allEvents = getAllEventsForDay(day);
                          
                          const eventsStartingHere = allEvents.filter(event => 
                            event.start >= hourStart && event.start < hourEnd
                          );

                          return (
                            <div
                              key={day}
                              onDragOver={(e) => e.preventDefault()}
                              onDrop={() => handleDrop(day, hour)}
                              className="border-r border-gray-200 last:border-r-0 relative"
                              style={{ minHeight: '60px' }}
                            >
                              {eventsStartingHere.map((event) => {
                                const eventDuration = event.end - event.start;
                                const eventHeight = (eventDuration / 60) * 60;
                                
                                const columnWidth = event.columnCount > 1 ? `${100 / event.columnCount}%` : '100%';
                                const leftPosition = event.columnCount > 1 ? `${(event.columnIndex / event.columnCount) * 100}%` : '0';
                                
                                return (
                                  <div
                                    key={event.uniqueId}
                                    className={`${getColorForEvent(event)} text-white rounded shadow-sm absolute group overflow-hidden flex flex-col`}
                                    style={{ 
                                      height: `${eventHeight}px`,
                                      top: `${((event.start % 60) / 60) * 60}px`,
                                      left: leftPosition,
                                      width: columnWidth,
                                      padding: '3px 4px',
                                      minHeight: '30px'
                                    }}
                                  >
                                    <div className="font-bold" style={{ 
                                      fontSize: eventHeight < 35 ? '8px' : '10px',
                                      lineHeight: eventHeight < 35 ? '9px' : '11px',
                                      marginBottom: '2px',
                                      textShadow: '0 1px 2px rgba(0,0,0,0.4)',
                                      wordBreak: 'break-word',
                                      overflow: 'hidden',
                                      display: '-webkit-box',
                                      WebkitLineClamp: eventHeight < 35 ? '2' : '3',
                                      WebkitBoxOrient: 'vertical'
                                    }}>
                                      {event.name}
                                    </div>
                                    
                                    <div style={{
                                      height: '0.5px',
                                      backgroundColor: 'rgba(255,255,255,0.5)',
                                      margin: '1px 0 2px 0',
                                      flexShrink: 0
                                    }} />
                                    
                                    <div className="opacity-95" style={{ 
                                      fontSize: eventHeight < 35 ? '7px' : '9px',
                                      lineHeight: eventHeight < 35 ? '8px' : '10px',
                                      fontWeight: '500',
                                      textShadow: '0 1px 1px rgba(0,0,0,0.3)',
                                      overflow: 'hidden',
                                      wordBreak: 'break-word'
                                    }}>
                                      {formatTime(event.start).replace(' ', '')}-{formatTime(event.end).replace(' ', '')}
                                    </div>
                                    
                                    {eventHeight >= 50 && (
                                      <div className="opacity-80" style={{ 
                                        fontSize: '8px',
                                        lineHeight: '9px',
                                        fontWeight: '400'
                                      }}>
                                        {eventDuration} min
                                      </div>
                                    )}
                                    
                                    {!event.isFixed && (
                                      <button
                                        onClick={() => removeSlot(event.taskId, event.slotIndex)}
                                        className="absolute top-0.5 right-0.5 opacity-0 group-hover:opacity-100 text-white hover:bg-white hover:bg-opacity-30 rounded p-0.5 transition-opacity"
                                      >
                                        <Trash2 size={8} />
                                      </button>
                                    )}
                                  </div>
                                );
                              })}
                            </div>
                          );
                        })}
                      </div>
                    );
                  })}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default UnifiedScheduleApp;
