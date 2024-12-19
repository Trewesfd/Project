# Project
Проект представляет собой простой HTTP-сервер на языке Go
package main

import (
"encoding/json"
"errors"
"fmt"
"net/http"
"strconv"
"strings"
)

type CalculateRequest struct {
Expression string json:"expression"
}

type CalculateResponse struct {
Result float64 json:"result"
Error  string  json:"error"
}

func CalculateHandler(w http.ResponseWriter, r *http.Request) {
if r.Method != http.MethodPost {
http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
return
}
var req CalculateRequest
err := json.NewDecoder(r.Body).Decode(&req)
if err != nil {
	http.Error(w, err.Error(), http.StatusBadRequest)
	return
}

result, err := Calc(req.Expression)
if err != nil {
	if err.Error() == "Expression is not valid" {
		resp := CalculateResponse{Result: 0, Error: "Expression is not valid"}
		w.WriteHeader(http.StatusUnprocessableEntity)
		json.NewEncoder(w).Encode(resp)
		return
	} else {
		resp := CalculateResponse{Result: 0, Error: "Internal server error"}
		w.WriteHeader(http.StatusInternalServerError)
		json.NewEncoder(w).Encode(resp)
		return
	}
}

resp := CalculateResponse{Result: result, Error: ""}
w.WriteHeader(http.StatusOK)
json.NewEncoder(w).Encode(resp)
}

func Calc(expression string) (float64, error) {
tokens := tokenize(expression)
postfix, err := infixToPostfix(tokens)
if err != nil {
return 0, err
}
return evaluatePostfix(postfix)
}

// код функций tokenize, infixToPostfix, evaluatePostfix, isNumber, isOperator, precedence остаётся как в предыдущем примере

func main() {
http.HandleFunc("/api/v1/calculate", CalculateHandler)
fmt.Println("Server is running on port 8080")
http.ListenAndServe(":8080", nil)
}
