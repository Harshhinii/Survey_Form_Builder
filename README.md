# Survey Form Builder (ReactJS)
## Date:12-05-2025

## AIM
To create a Survey Form Builder using ReactJS..

## ALGORITHM
### STEP 1 Initialize States
mode ← 'build' (default mode)

questions ← [] (holds all survey questions)

currentQuestion ← { text: '', type: 'text', options: '' } (holds input while building)

editingIndex ← null (tracks which question is being edited)

responses ← {} (user responses to survey)

submitted ← false (indicates whether survey was submitted)

### STEP 2 Switch Modes
Toggle mode between 'build' and 'fill' when the toggle button is clicked.

Reset submitted to false when entering Fill Mode.

### STEP 3 Build Mode Logic
#### a. Add/Update Question
If currentQuestion.text is not empty:

If type is "radio" or "checkbox", split currentQuestion.options by commas → optionsArray.

Construct a questionObject:

If editingIndex is not null, replace question at that index.

Else, push questionObject into questions.

Reset currentQuestion and editingIndex.

#### b. Edit Question
Set currentQuestion to the selected question’s data.

Set editingIndex to the question’s index.

#### c. Delete Question
Remove the selected question from questions.

### STEP 4 Fill Mode Logic
#### a. Render Form
For each question in questions:

If type is "text", render a text input.

If type is "radio", render radio buttons for each option.

If type is "checkbox", render checkboxes.

#### b. Capture Responses
On input change:

For "text" and "radio", update responses[index] = value.

For "checkbox":

If option exists in responses[index], remove it.

Else, add it to the array.

### STEP 5 Submit Form
When user clicks "Submit":

Set submitted = true.

Display a summary using the questions and responses.

### STEP 6 Display Summary
Iterate over questions.

Display the response from responses[i] for each question:

If it's an array, join it with commas.

If empty, show "No response".


## PROGRAM
app.jsx
```
import React, { useState } from 'react';
import './App.css';

function App() {
  const [mode, setMode] = useState('build');
  const [questions, setQuestions] = useState([]);
  const [currentQuestion, setCurrentQuestion] = useState({ text: '', type: 'text', options: '' });
  const [editingIndex, setEditingIndex] = useState(null);
  const [responses, setResponses] = useState({});
  const [submitted, setSubmitted] = useState(false);

  const toggleMode = () => {
    setMode(mode === 'build' ? 'fill' : 'build');
    setSubmitted(false);
  };

  const handleAddQuestion = () => {
    if (!currentQuestion.text.trim()) return;

    const newQuestion = {
      text: currentQuestion.text,
      type: currentQuestion.type,
      options: ['radio', 'checkbox'].includes(currentQuestion.type)
        ? currentQuestion.options.split(',').map(o => o.trim())
        : []
    };

    if (editingIndex !== null) {
      const updated = [...questions];
      updated[editingIndex] = newQuestion;
      setQuestions(updated);
    } else {
      setQuestions([...questions, newQuestion]);
    }

    setCurrentQuestion({ text: '', type: 'text', options: '' });
    setEditingIndex(null);
  };

  const handleEdit = (index) => {
    const q = questions[index];
    setCurrentQuestion({
      text: q.text,
      type: q.type,
      options: q.options.join(', ')
    });
    setEditingIndex(index);
  };

  const handleDelete = (index) => {
    const updated = questions.filter((_, i) => i !== index);
    setQuestions(updated);
  };

  const handleResponseChange = (index, value, type, option = null) => {
    if (type === 'text' || type === 'radio') {
      setResponses({ ...responses, [index]: value });
    } else if (type === 'checkbox') {
      const existing = responses[index] || [];
      const updated = existing.includes(option)
        ? existing.filter(o => o !== option)
        : [...existing, option];
      setResponses({ ...responses, [index]: updated });
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    setSubmitted(true);
  };

  return (
    <div className="App">
      <button onClick={toggleMode}>
        Switch to {mode === 'build' ? 'Fill' : 'Build'} Mode
      </button>
      <h1>Placement Training </h1>

      {mode === 'build' && (
        <div className="build-section">
          <input
            type="text"
            placeholder="Enter question text"
            value={currentQuestion.text}
            onChange={(e) => setCurrentQuestion({ ...currentQuestion, text: e.target.value })}
          />
          <select
            value={currentQuestion.type}
            onChange={(e) => setCurrentQuestion({ ...currentQuestion, type: e.target.value })}
          >
            <option value="text">Text</option>
            <option value="radio">Multiple Choice (Radio)</option>
            <option value="checkbox">Multiple Choice (Checkbox)</option>
          </select>
          {['radio', 'checkbox'].includes(currentQuestion.type) && (
            <input
              type="text"
              placeholder="Enter comma-separated options"
              value={currentQuestion.options}
              onChange={(e) => setCurrentQuestion({ ...currentQuestion, options: e.target.value })}
            />
          )}
          <button onClick={handleAddQuestion}>
            {editingIndex !== null ? 'Update Question' : 'Add Question'}
          </button>

          <ul>
            {questions.map((q, index) => (
              <li key={index}>
                {q.text} ({q.type})
                <button onClick={() => handleEdit(index)}>Edit</button>
                <button onClick={() => handleDelete(index)}>Delete</button>
              </li>
            ))}
          </ul>
        </div>
      )}

      {mode === 'fill' && (
        <form onSubmit={handleSubmit}>
          {questions.map((q, index) => (
            <div key={index} className="question-block">
              <p>{q.text}</p>
              {q.type === 'text' && (
                <input
                  type="text"
                  value={responses[index] || ''}
                  onChange={(e) => handleResponseChange(index, e.target.value, 'text')}
                />
              )}
              {q.type === 'radio' &&
                q.options.map((option, i) => (
                  <div key={i}>
                    <input
                      type="radio"
                      name={q-${index}}
                      value={option}
                      checked={responses[index] === option}
                      onChange={(e) => handleResponseChange(index, e.target.value, 'radio')}
                    />
                    {option}
                  </div>
                ))}
              {q.type === 'checkbox' &&
                q.options.map((option, i) => (
                  <div key={i}>
                    <input
                      type="checkbox"
                      name={q-${index}-${option}}
                      value={option}
                      checked={(responses[index] || []).includes(option)}
                      onChange={() => handleResponseChange(index, null, 'checkbox', option)}
                    />
                    {option}
                  </div>
                ))}
            </div>
          ))}
          <button type="submit">Submit</button>
        </form>
      )}

      {submitted && (
        <div className="summary">
          <h2>Survey Summary</h2>
          {questions.map((q, index) => (
            <p key={index}>
              <strong>{q.text}</strong><br />
              {Array.isArray(responses[index])
                ? responses[index].join(', ')
                : responses[index] || 'No response'}
            </p>
          ))}
        </div>
      )}
    </div>
  );
}

export default App;
```
app.css
```
.App {
  font-family: Arial, sans-serif;
  padding: 2rem;
  min-height: 100vh;
  background: linear-gradient(to bottom right, #a2c5f9, #ffffff);
}

button {
  margin-bottom: 1rem;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  border: 1px solid #333;
  background-color: #f5f5f5;
  cursor: pointer;
}

button:hover {
  background-color: #d6f1ed;
}

.question-block {
  background: linear-gradient(to bottom, #b6ebed, #fff);
  border: 1px solid #c6ecf0;
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 1.5rem;
}

input[type="text"] {
  width: 100%;
  padding: 0.5rem;
  font-size: 1rem;
}

.summary {
  margin-top: 2rem;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  padding: 1rem;
  border-radius: 8px;
}
```

## OUTPUT

![screenshot 1](https://github.com/user-attachments/assets/6e2cb670-0f7f-47fe-9dbe-d4cf618089e9)
![screenshot 2](https://github.com/user-attachments/assets/f6d4a657-0c79-439a-84e5-eeb112d5e955)

## RESULT
The program for creating Survey Form Builder using ReactJS is executed successfully.
